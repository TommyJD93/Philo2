<div align='center'> <h1>Philosophers</h1> </div>
<p align='center'>I never thought philosophy would be so deadly</p>
<p align='center'>Dining philosophers problem's solution for 42 cursus project</p>
<div align='center'> <h1>General idea</h1> </div>
The mandatory part of this project asks us to solve the <a href='https://en.wikipedia.org/wiki/Dining_philosophers_problem'>dining philosophers problem</a> and 
implement a mutithreading solution. In order to better understand the solution that we are going to implement in this project I suggest you to read something about what
a thread is and how multithreading works, I'll leave a couple of wikipedia references to start learning about these topics:
<ul>
<li><a href='https://en.wikipedia.org/wiki/Thread_(computing)'>Thread</a></li>
<li><a href='https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)'>Multi Threading</a></li>
</ul>
Another raccomandation is to read the subject before starting, I'll leave a link also to that: <a href='https://cdn.intra.42.fr/pdf/pdf/51354/en.subject.pdf'>subject</a>.<br>
Now that we know what we have to do we can start explaining the general idea that I've applied in this project. First of all we have to immagine a round table, N num of 
philosophers sits around it and each of them brings a fork and let's say that they place it on the table on their right (doesn't really change if they place it on the right or left).
At this point we know that a philosopher can do three things: eat, think or sleep; but in order to eat he has to pick two forks (the one on his right and the one on his left).
Let's use a picture to have a more concrete idea of what we are talking about:
<br><br>
<a href='https://en.wikipedia.org/wiki/Dining_philosophers_problem#/media/File:An_illustration_of_the_dining_philosophers_problem.png'><img src='https://user-images.githubusercontent.com/59456000/198906008-4135d5d2-de53-4a8b-9c24-744181f04851.png' width='275' heigth='275'></img></a>
<br><br>
Since I'm not a big fan of philosophy i don't know a single name one these guys up here so I'll give them some more familiar names and starting from the bottom left
one they will be: Roberto Legno, Thiago, Marcello, Lapo Raspanti and Rekkless.<br>
<p>Let's say that Roberto Legno wants to eat, so he picks his right and left fork, at this point we notice that Rekkless can't eat since Roberto Legno picked his right fork witch was shared with Rekkless; this might seem a little obvious but keep in mind this situation because the main problem of this project is how to organize the eating action of the philosophers.<br>
Probably the first solution that came to your mind is to simply make the odd and even philos eat separately, well we are not going to do that, it's too hard coded and we would loose the meaning of the project, <i><b>philos have to organize by themselves.</b></i></br>
But, how are we going to do that? Using mutex!</p></p>
<div align='center'> <h1>Race Conditions & Mutexe</h1> </div>
<h3>Race conditions</h3>
Before explaining what mutex are and why we have to use them, let's talk about what race conditions are. A <a href='https://stackoverflow.com/questions/34510/what-is-a-race-condition'>Race condition</a> it is a condition in which one or more threads are trying to access and modify a same variable at the same time, this can lead to an error in the final value of that variable. To better understan the race condition here's an example:
Let's say that we want to count to 2.000.000, to do that with the multithreading we simply make two threads that execute the same routine, and the routine increase the variable cont to 1.000.000, in this way we should execute the while inside routine 2 times and when cont is printed we should get 2.000.000. Well, that's not exactly how it works.
<pre>
<code>
#include <pthread.h>
#include <stdio.h>

int cont = 0;

void  *routine()
{
  int i;

  i = -1;
  while (++i < 1000000)
  	  cont++;
  return (NULL);
}

int main()
{
  pthread_t tid1, tid2;

  pthread_create(&tid1, NULL, &routine, NULL);
  pthread_create(&tid2, NULL, &routine, NULL);

  pthread_join(tid1, NULL);
  pthread_join(tid2, NULL);
  printf("cont: %d\n", cont);
}
</code>
</pre>
If you try to execute the code above you will see that you'll never get 2.000.000 that's because a race condition is happening. To better understand what's happening let's take a look at the assembly of the "cont++" instruction.
To increase a variable by 1 the assembly execute 3 operations:
- read, simply get the variable value 
- increase, increment locally the variable
- write, updates the value of the variable
<br>
Assume that the current value of cont is 23, let's see what the assembly would do:
<pre>
<code>
read: 23
increase: 23
write: 24
</code>
</pre>
That happens when we simply do cont++ in a non multithreaded program, let's see what happens in a mutithreaded program when a race condition happens:
<pre>
<code>
--------------------------------
|   Thread A   |   Thread B    |
| read:   23   | read:   23    |
| increase: 23 | increase:     |
| write:   24  | write:        |
--------------------------------
</code>
</pre>
At this point the thread B is paused, because is doing the same operation as A and it's restarted after a while, but while B is paused A continue to increase the cont value (for the example we say it reaches 30). Let's se what happens when B restart: 
<pre>
<code>
--------------------------------
|   Thread A   |   Thread B    |
| read:   30   | read:   23    |
| increase: 30 | increase:  23 |
| write:   31  | write:   24   |
--------------------------------
</code>
</pre>
As we can see B restart from where he was paused, since he already read 23 he won't read the current value of cont, he will keep doing his operation with the last value he read before stopping, in this case it's 23 so he will update the cont value to 24 and therefore A's next iteration won't read 31 but 24.
<h3>Mutex</h3>
