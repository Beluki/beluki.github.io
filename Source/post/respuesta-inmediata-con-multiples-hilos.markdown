title: Respuesta inmediata con múltiples hilos
date: 2013-09-04
tags: ['programacion', 'python']

La idea básica de [MultiHash][] consiste en calcular múltiples algoritmos,
como md5, sha1, etc..., de múltiples archivos, leyendo cada archivo
una sola vez.

[MultiHash]: https://github.com/Beluki/MultiHash

Aunque ésto ya mejora la eficiencia notablemente (con respecto a las
herramientas en [coreutils][]), podemos hacerlo mejor. La idea es dividir
el trabajo en tareas, donde cada tarea calcula todos los algoritmos para
un archivo. Una serie de hilos ejecuta estas tareas a la vez. En un equipo
con 2 o más procesadores y un disco duro rápido ([SSD][], [RAID][])
la diferencia de velocidad es considerable.

[coreutils]: http://www.gnu.org/software/coreutils
[SSD]: http://es.wikipedia.org/wiki/SSD
[RAID]: http://es.wikipedia.org/wiki/RAID

Un problema interesante cuando se trabaja con hilos (aparte de sincronización)
es cómo presentar los resultados al usuario, tan pronto como estén listos y
en el orden correcto, independientemente de los hilos que acaben antes o que
aún se estén ejecutando. El objetivo, desde un punto de vista de interfaz, es
ocultar el hecho de que se están usando hilos, que debería ser un detalle
interno, y dar la impresión de un programa secuencial normal.

El resto de este post trata sobre cómo hacerlo en Python. Es esencialmente el
mismo código que usa MultiHash, simplificado a modo de ejemplo.

## Hilos y tareas

El primer paso es obvio, crear una subclase de Thread que toma tareas de una
Queue, las ejecuta y añade el resultado (que se guarda en la propia tarea)
a otra Queue. Entenderemos como tarea cualquier objeto que implemente un
método "run()".

    ::python
    class Worker(Thread):
        def __init__(self, todo, done):
            super().__init__()
            self.todo = todo
            self.done = done
            self.daemon = True
            self.start()

        def run(self):
            while True:
                task = self.todo.get()
                task.run()
                self.done.put(task)
                self.todo.task_done()

Las tareas simplemente calcularán números de fibonacci usando un algoritmo
recursivo (y terriblemente ineficiente) guardando el resultado en una
propiedad de la propia tarea:

    ::python
    def fib(n):
        if n < 2:
            return 1
        else:
            return fib(n - 1) + fib(n - 2)


    class FibTask(object):

        def __init__(self, number):
            self.number = number

        def run(self):
            self.result = fib(self.number)

Finalmente, un ThreadPool empezará a ejecutar tareas usando tantos hilos
como le pidamos al inicializar:

    ::python
    class ThreadPool(object):

        def __init__(self, threads):
            self.threads = threads

            self.tasks = []
            self.results = set()

            self.todo = Queue()
            self.done = Queue()

        def start(self, tasks):
            """ Start computing tasks. """
            self.tasks = tasks

            for task in self.tasks:
                self.todo.put(task)

            for x in range(self.threads):
                Worker(self.todo, self.done)

## Esperando a las tareas terminadas

Aquí viene la parte interesante. El objeto que representa cada tarea
puede ser su propio identificador. La propiedad ".tasks" de ThreadPool
mantiene el orden inicial de las tareas. La propiedad ".results" contendrá
las tareas completadas.

Podemos añadir un método que itera el orden inicial y devuelve cada
tarea completada tan pronto como se encuentre en los resultados:

    ::python
    class ThreadPool(object):
        ...

        def poll_completed_tasks(self):
            """
            Yield the computed tasks, in the order specified when 'start(tasks)'
            was called, as soon as they are finished.
            """
            for task in self.tasks:
                while True:
                    if task in self.results:
                        yield task
                        break
                    else:
                        self.wait_for_task()

            # at this point, all the tasks are completed:
            self.todo.join()

Donde wait_for_task() está implementado de la siguiente manera:

    ::python
    class ThreadPool(object):
        ...

        def wait_for_task(self):
            """ Wait for one task to complete. """
            while True:
                try:
                    task = self.done.get(block = False)
                    self.results.add(task)
                    break

                # give tasks processor time:
                except queue.Empty:
                    time.sleep(0.1)

Intentamos tomar una tarea completada, sin bloquear. Si no lo conseguimos,
dormimos durante 0.1 segundos y lo intentamos de nuevo, hasta que una tarea
ha sido completada.

Ejemplo de uso:

    ::python
    def main():
        cpus = cpu_count()
        pool = ThreadPool(cpus)

        tasks = [FibTask(n) for n in range(1, 33)]
        tasks += [FibTask(n) for n in reversed(range(1, 33))]

        pool.start(tasks)

        # should print the results in order
        # first from 1 to 32, then from 32 to 1:
        for task in pool.poll_completed_tasks():
            print('fib({0.number}): {0.result}'.format(task), flush = True)


    if __name__ == '__main__':
        try:
            main()
        except KeyboardInterrupt:
            pass

Aunque algunas tareas terminen antes que otras, los resultados se mostrarán
en orden, tan pronto como sea factible. Control + C debe detener la ejecución
correctamente.

El código completo del ejemplo está disponible [aquí](https://github.com/Beluki/Misc/blob/master/Source/ThreadPool/ThreadPool.py).

