##Creating and running a thread

- Main.java

```java
/**
 *  Main class of the example
 */
public class Main {

	/**
	 * Main method of the example
	 * @param args
	 */
	public static void main(String[] args) {

		//Launch 10 threads that make the operation with a different number
		for (int i=1; i<=10; i++){
			Calculator calculator=new Calculator(i);
			Thread thread=new Thread(calculator);
			thread.start();
		}
	}
}
```

- Calculator.java

```java
/**
 *  This class prints the multiplication table of a number
 */
public class Calculator implements Runnable {

	/**
	 *  The number
	 */
	private int number;
	
	/**
	 *  Constructor of the class
	 * @param number : The number
	 */
	public Calculator(int number) {
		this.number=number;
	}
	
	/**
	 *  Method that do the calculations
	 */
	@Override
	public void run() {
		for (int i=1; i<=10; i++){
			System.out.printf("%s: %d * %d = %d\n",Thread.currentThread().getName(),number,i,i*number);
		}
	}

}
```
- 输出结果: 运行了线程

--------------------------

## Getting and setting thread information

- 改变Main.java文件,Calculator.java文件不变
```
/**
 *  Main class of the example
 */
public class Main {

	/**
	 * Main method of the example
	 * @param args
	 */
	public static void main(String[] args) {

		// Thread priority infomation 
		System.out.printf("Minimum Priority: %s\n",Thread.MIN_PRIORITY);
		System.out.printf("Normal Priority: %s\n",Thread.NORM_PRIORITY);
		System.out.printf("Maximun Priority: %s\n",Thread.MAX_PRIORITY);
		
		Thread threads[];
		Thread.State status[];
		
		// Launch 10 threads to do the operation, 5 with the max
		// priority, 5 with the min
		threads=new Thread[10];
		status=new Thread.State[10];
		for (int i=0; i<10; i++){
			threads[i]=new Thread(new Calculator(i));
			if ((i%2)==0){
				threads[i].setPriority(Thread.MAX_PRIORITY);
			} else {
				threads[i].setPriority(Thread.MIN_PRIORITY);
			}
			threads[i].setName("Thread "+i);
		}
		
		
		// Wait for the finalization of the threads. Meanwhile, 
		// write the status of those threads in a file
		try (FileWriter file = new FileWriter(".\\data\\log.txt");PrintWriter pw = new PrintWriter(file);){
			
			for (int i=0; i<10; i++){
				pw.println("Main : Status of Thread "+i+" : "+threads[i].getState());
				status[i]=threads[i].getState();
			}

			for (int i=0; i<10; i++){
				threads[i].start();
			}
			
			boolean finish=false;
			while (!finish) {
				for (int i=0; i<10; i++){
					if (threads[i].getState()!=status[i]) {
						writeThreadInfo(pw, threads[i],status[i]);
						status[i]=threads[i].getState();
					}
				}
				
				finish=true;
				for (int i=0; i<10; i++){
					finish=finish &&(threads[i].getState()==State.TERMINATED);
				}
			}
			
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	/**
	 *  This method writes the state of a thread in a file
	 * @param pw : PrintWriter to write the data
	 * @param thread : Thread whose information will be written
	 * @param state : Old state of the thread
	 */
	private static void writeThreadInfo(PrintWriter pw, Thread thread, State state) {
		pw.printf("Main : Id %d - %s\n",thread.getId(),thread.getName());
		pw.printf("Main : Priority: %d\n",thread.getPriority());
		pw.printf("Main : Old State: %s\n",state);
		pw.printf("Main : New State: %s\n",thread.getState());
		pw.printf("Main : ************************************\n");
	}

}
```

- 输出结果:log.txt文件中会看到10个线程的各自的id,名称,权限,状态变化.具体化的展现了线程的变化

## Interrupting a thread

- Main.java
```java
/**
 *  Main class of the sample. Launch the PrimeGenerator, waits 
 *  five seconds and interrupts the Thread
 */
public class Main {

	/**
	 * Main method of the sample. Launch the PrimeGenerator, waits
	 * five seconds and interrupts the Thread
	 * @param args
	 */
	public static void main(String[] args) {

		// Launch the prime numbers generator
		Thread task=new PrimeGenerator();
		task.start();
		
		// Wait 5 seconds
		try {
			TimeUnit.SECONDS.sleep(5);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		// Interrupt the prime number generator
		task.interrupt();
	}

}
```

- PrimeGenerator.java

```java
/**
 *  This class generates prime numbers until is interrumped
 */
public class PrimeGenerator extends Thread{

	/**
	 *  Central method of the class
	 */
	@Override
	public void run() {
		long number=1L;
		
		// This bucle never ends... until is interrupted
		while (true) {
			if (isPrime(number)) {
				System.out.printf("Number %d is Prime\n",number);
			}
			
			// When is interrupted, write a message and ends
			if (isInterrupted()) {
				System.out.printf("The Prime Generator has been Interrupted\n");
				return;
			}
			number++;
		}
	}

	/**
	 *  Method that calculate if a number is prime or not
	 * @param number : The number
	 * @return A boolean value. True if the number is prime, false if not.
	 */
	private boolean isPrime(long number) {
		if (number <=2) {
			return true;
		}
		for (long i=2; i<number; i++){
			if ((number % i)==0) {
				return false;
			}
		}
		return true;
	}

}
```

- 输出结果: 一直打印奇数,知道线程会自己结束

- isInterrupted() vs interrupted()方法的区别?


## Controlling the interruption of a thread

- Main.java

```java
/**
 *  Main class of the example. Search for the autoexect.bat file
 *  on the Windows root folder and its subfolders during ten seconds
 *  and then, interrupts the Thread
 */
public class Main {

	/**
	 * Main method of the core. Search for the autoexect.bat file
	 * on the Windows root folder and its subfolders during ten seconds
	 * and then, interrupts the Thread
	 * @param args
	 */
	public static void main(String[] args) {
		// Creates the Runnable object and the Thread to run it
		FileSearch searcher=new FileSearch("C:\\","autoexec.bat");
		Thread thread=new Thread(searcher);
		
		// Starts the Thread
		thread.start();
		
		// Wait for ten seconds
		try {
			TimeUnit.SECONDS.sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		// Interrupts the thread
		thread.interrupt();
	}

}
```

- FileSearch.java

```java
/**
 * This class search for files with a name in a directory
 */
public class FileSearch implements Runnable {

	/**
	 * Initial path for the search
	 */
	private String initPath;
	/**
	 * Name of the file we are searching for
	 */
	private String fileName;

	/**
	 * Constructor of the class
	 * 
	 * @param initPath
	 *            : Initial path for the search
	 * @param fileName
	 *            : Name of the file we are searching for
	 */
	public FileSearch(String initPath, String fileName) {
		this.initPath = initPath;
		this.fileName = fileName;
	}

	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		File file = new File(initPath);
		if (file.isDirectory()) {
			try {
				directoryProcess(file);
			} catch (InterruptedException e) {
				System.out.printf("%s: The search has been interrupted",Thread.currentThread().getName());
				cleanResources();
			}
		}
	}

	/**
	 * Method for cleaning the resources. In this case, is empty
	 */
	private void cleanResources() {

	}

	/**
	 * Method that process a directory
	 * 
	 * @param file
	 *            : Directory to process
	 * @throws InterruptedException
	 *             : If the thread is interrupted
	 */
	private void directoryProcess(File file) throws InterruptedException {

		// Get the content of the directory
		File list[] = file.listFiles();
		if (list != null) {
			for (int i = 0; i < list.length; i++) {
				if (list[i].isDirectory()) {
					// If is a directory, process it
					directoryProcess(list[i]);
				} else {
					// If is a file, process it
					fileProcess(list[i]);
				}
			}
		}
		// Check the interruption
		if (Thread.interrupted()) {
			throw new InterruptedException();
		}
	}

	/**
	 * Method that process a File
	 * 
	 * @param file
	 *            : File to process
	 * @throws InterruptedException
	 *             : If the thread is interrupted
	 */
	private void fileProcess(File file) throws InterruptedException {
		// Check the name
		if (file.getName().equals(fileName)) {
			System.out.printf("%s : %s\n",Thread.currentThread().getName() ,file.getAbsolutePath());
		}

		// Check the interruption
		if (Thread.interrupted()) {
			throw new InterruptedException();
		}
	}

}
```

- 输出结果: 线程停止的时候抛出异常,优雅地控制Interruption异常

## Sleeping and resuming a thread

- Main.java

```java
/**
 * Main class of the Example. Creates a FileClock runnable object
 * and a Thread to run it. Starts the Thread, waits five seconds
 * and interrupts it. 
 *
 */
public class Main {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// Creates a FileClock runnable object and a Thread
		// to run it
		FileClock clock=new FileClock();
		Thread thread=new Thread(clock);
		
		// Starts the Thread
		thread.start();
		try {
			// Waits five seconds
			TimeUnit.SECONDS.sleep(5);
		} catch (InterruptedException e) {
			e.printStackTrace();
		};
		// Interrupts the Thread
		thread.interrupt();
	}
}

```

- FileClock.java

```java
/**
 * Class that writes the actual date to a file every second
 * 
 */
public class FileClock implements Runnable {

	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.printf("%s\n", new Date());
			try {
				// Sleep during one second
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				System.out.printf("The FileClock has been interrupted");
			}
		}
	}
}
```

- 输出结果: 输出10秒,线程会在任意一秒钟停止,但是会继续输出余下的时间.原因在于线程sleep()的时候不占用cpu,此时cpu就可以执行其他的任务

- yield()方法 vs sleep()?

同样可以不占用cpu,但是JVM不保证会遵守,所以这个方法一般用于调试目的.

## Waiting for the finalization of a thread

- Main.java

```java
/**
 * Main class of the Example. Create and start two initialization tasks
 * and wait for their finish
 *
 */
public class Main {

	/**
	 * Main method of the class. Create and star two initialization tasks
	 * and wait for their finish
	 * @param args
	 */
	public static void main(String[] args) {

		// Creates and starts a DataSourceLoader runnable object
		DataSourcesLoader dsLoader = new DataSourcesLoader();
		Thread thread1 = new Thread(dsLoader,"DataSourceThread");
		thread1.start();

		// Creates and starts a NetworkConnectionsLoader runnable object
		NetworkConnectionsLoader ncLoader = new NetworkConnectionsLoader();
		Thread thread2 = new Thread(ncLoader,"NetworkConnectionLoader");
		thread2.start();

		// Wait for the finalization of the two threads
		try {
			thread1.join();
			thread2.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

		// Waits a message
		System.out.printf("Main: Configuration has been loaded: %s\n",new Date());
	}
}
```

- DataSourcesLoader.java

```java
/**
 * Class that simulates an initialization operation. It sleeps during four seconds
 *
 */
public class DataSourcesLoader implements Runnable {


	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		
		// Writes a messsage
		System.out.printf("Begining data sources loading: %s\n",new Date());
		// Sleeps four seconds
		try {
			TimeUnit.SECONDS.sleep(4);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		// Writes a message
		System.out.printf("Data sources loading has finished: %s\n",new Date());
	}
}
```

- NetworkConnectionsLoader.java

```java
/**
 * Class that simulates an initialization operation. It sleeps during six seconds 
 *
 */
public class NetworkConnectionsLoader implements Runnable {


	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		// Writes a message
		System.out.printf("Begining network connections loading: %s\n",new Date());
		// Sleep six seconds
		try {
			TimeUnit.SECONDS.sleep(6);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		// Writes a message
		System.out.printf("Network connections loading has finished: %s\n",new Date());
	}
}
```

- 输出结果: 重点理解join()方法,可以演示 数据加载和网络连接结束后,在执行Main方法中的语句s

## Creating and running a daemon thread

- Main.java

```java
/**
 * Main class of the example. Creates three WriterTaks and a CleanerTask 
 *
 */
public class Main {

	/**
	 * Main method of the example. Creates three WriterTasks and a CleanerTask
	 * @param args
	 */
	public static void main(String[] args) {
		
		// Creates the Event data structure
		Deque<Event> deque=new ArrayDeque<Event>();
		
		// Creates the three WriterTask and starts them
		WriterTask writer=new WriterTask(deque);
		for (int i=0; i<3; i++){
			Thread thread=new Thread(writer);
			thread.start();
		}
		
		// Creates a cleaner task and starts them
		CleanerTask cleaner=new CleanerTask(deque);
		cleaner.start();

	}

}
```

- Event.java
```java
/**
 * Class that stores event's information 
 *
 */
public class Event {

	/**
	 * Date of the event
	 */
	private Date date;
	
	/**
	 * Message of the event
	 */
	private String event;
	
	/**
	 * Reads the Date of the event
	 * @return the Date of the event
	 */
	public Date getDate() {
		return date;
	}
	
	/**
	 * Writes the Date of the event
	 * @param date the date of the event
	 */
	public void setDate(Date date) {
		this.date = date;
	}
	
	/**
	 * Reads the message of the event
	 * @return the message of the event
	 */
	public String getEvent() {
		return event;
	}
	
	/**
	 * Writes the message of the event
	 * @param event the message of the event
	 */
	public void setEvent(String event) {
		this.event = event;
	}
}
```

- CleanerTask.java

```java
/**
 * Class that review the Event data structure and delete
 * the events older than ten seconds
 *
 */
public class CleanerTask extends Thread {

	/**
	 * Data structure that stores events
	 */
	private Deque<Event> deque;

	/**
	 * Constructor of the class
	 * @param deque data structure that stores events
	 */
	public CleanerTask(Deque<Event> deque) {
		this.deque = deque;
		// Establish that this is a Daemon Thread
		setDaemon(true);
	}


	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		while (true) {
			Date date = new Date();
			clean(date);
		}
	}

	/**
	 * Method that review the Events data structure and delete
	 * the events older than ten seconds
	 * @param date
	 */
	private void clean(Date date) {
		long difference;
		boolean delete;
		
		if (deque.size()==0) {
			return;
		}
		
		delete=false;
		do {
			Event e = deque.getLast();
			difference = date.getTime() - e.getDate().getTime();
			if (difference > 10000) {
				System.out.printf("Cleaner: %s\n",e.getEvent());
				deque.removeLast();
				delete=true;
			}	
		} while (difference > 10000);
		if (delete){
			System.out.printf("Cleaner: Size of the queue: %d\n",deque.size());
		}
	}
}
```

- WriterTask.java

```java
/**
 * Runnable class that generates and event every second
 *
 */
public class WriterTask implements Runnable {

	/**
	 * Data structure to stores the events
	 */
	Deque<Event> deque;
	
	/**
	 * Constructor of the class
	 * @param deque data structure that stores the event
	 */
	public WriterTask (Deque<Event> deque){
		this.deque=deque;
	}
	
	/**
	 * Main class of the Runnable
	 */
	@Override
	public void run() {
		
		// Writes 100 events
		for (int i=1; i<100; i++) {
			// Creates and initializes the Event objects 
			Event event=new Event();
			event.setDate(new Date());
			event.setEvent(String.format("The thread %s has generated an event",Thread.currentThread().getId()));
			
			// Add to the data structure
			deque.addFirst(event);
			try {
				// Sleeps during one second
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

- 输出结果:  后台线程,权限比较低,典型的示例就是java垃圾回收器.这里CleanerTask是后台线程,打印出现在容器中event的数量

## Processing uncontrolled exceptions in a thread

- Main.java

```java
/**
 * Main class of the example. Initialize a Thread to process the uncaught
 * exceptions and starts a Task object that always throws an exception 
 *
 */
public class Main {

	/**
	 * Main method of the example. Initialize a Thread to process the 
	 * uncaught exceptions and starts a Task object that always throws an
	 * exception 
	 * @param args
	 */
	public static void main(String[] args) {
		// Creates the Task
		Task task=new Task();
		// Creates the Thread
		Thread thread=new Thread(task);
		// Sets de uncaugh exceptio handler
		thread.setUncaughtExceptionHandler(new ExceptionHandler());
		// Starts the Thread
		thread.start();
		
		try {
			thread.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		System.out.printf("Thread has finished\n");

	}

}
```

- Task.java

```java
/**
 * Runnable class than throws and Exception
 *
 */
public class Task implements Runnable {


	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		// The next instruction always throws and exception
		int numero=Integer.parseInt("TTT");
	}

}
```

- ExceptionHandler.java

```java
/**
 * Class that process the uncaught exceptions throwed in a Thread
 *
 */
public class ExceptionHandler implements UncaughtExceptionHandler {


	/**
	 * Main method of the class. It process the uncaught excpetions throwed
	 * in a Thread
	 * @param t The Thead than throws the Exception
	 * @param e The Exception throwed
	 */
	@Override	
	public void uncaughtException(Thread t, Throwable e) {
		System.out.printf("An exception has been captured\n");
		System.out.printf("Thread: %s\n",t.getId());
		System.out.printf("Exception: %s: %s\n",e.getClass().getName(),e.getMessage());
		System.out.printf("Stack Trace: \n");
		e.printStackTrace(System.out);
		System.out.printf("Thread status: %s\n",t.getState());
	}

}
```

- 输出结果: 创建ExceptionHandler类,来重写uncaughtException方法,依次来格式化输出异常信息,包括线程的状态


## Using local thread variables

- Main.java

```java
/**
 * Main class of the UnsafeTask. Creates a Runnable task and
 * three Thread objects that run it.
 *
 */
public class Main {

	/**
	 * Main method of the UnsafeTaks. Creates a Runnable task and
	 * three Thread objects that run it.
	 * @param args
	 */
	public static void main(String[] args) {
		// Creates the unsafe task
		UnsafeTask task=new UnsafeTask();
		
		// Throw three Thread objects
		for (int i=0; i<3; i++){
			Thread thread=new Thread(task);
			thread.start();
			try {
				TimeUnit.SECONDS.sleep(2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

- SafeMain.java

```java
/**
 * Main class of the example.
 *
 */
public class SafeMain {

	/**
	 * Main method of the example
	 * @param args
	 */
	public static void main(String[] args) {
		// Creates a task
		SafeTask task=new SafeTask();
		
		// Creates and start three Thread objects for that Task
		for (int i=0; i<3; i++){
			Thread thread=new Thread(task);
			try {
				TimeUnit.SECONDS.sleep(2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			thread.start();
		}

	}

}
```

- SafeTask.java

```java
/**
 * Class that shows the usage of ThreadLocal variables to share
 * data between Thread objects
 *
 */
public class SafeTask implements Runnable {

	/**
	 * ThreadLocal shared between the Thread objects
	 */
	private static ThreadLocal<Date> startDate= new ThreadLocal<Date>() {
		protected Date initialValue(){
			return new Date();
		}
	};
	

	/**
	 * Main method of the class
	 */
	@Override
	public void run() {
		// Writes the start date
		System.out.printf("Starting Thread: %s : %s\n",Thread.currentThread().getId(),startDate.get());
		try {
			TimeUnit.SECONDS.sleep((int)Math.rint(Math.random()*10));
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		// Writes the start date
		System.out.printf("Thread Finished: %s : %s\n",Thread.currentThread().getId(),startDate.get());
	}

}

```

- UnsafeTask.java

```java
/**
 * Class that shows the problem generate when some Thread objects
 * share a data structure
 *
 */
public class UnsafeTask implements Runnable{

	/**
	 * Date shared by all threads
	 */
	private Date startDate;
	
	/**
	 * Main method of the class. Saves the start date and writes
	 * it to the console when it starts and when it ends
	 */
	@Override
	public void run() {
		startDate=new Date();
		System.out.printf("Starting Thread: %s : %s\n",Thread.currentThread().getId(),startDate);
		try {
			TimeUnit.SECONDS.sleep((int)Math.rint(Math.random()*10));
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.printf("Thread Finished: %s : %s\n",Thread.currentThread().getId(),startDate);
	}

}

```

- 输出结果: 注意SafeTask中有ThreadLocal定义,这个是关键


## Grouping threads into a group

- Main.java

```java
public class Main {

	/**
	 * Main class of the example
	 * @param args
	 */
	public static void main(String[] args) {

		// Create a ThreadGroup
		ThreadGroup threadGroup = new ThreadGroup("Searcher");
		Result result=new Result();

		// Create a SeachTask and 10 Thread objects with this Runnable
		SearchTask searchTask=new SearchTask(result);
		for (int i=0; i<5; i++) {
			Thread thread=new Thread(threadGroup, searchTask);
			thread.start();
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		// Write information about the ThreadGroup to the console
		System.out.printf("Number of Threads: %d\n",threadGroup.activeCount());
		System.out.printf("Information about the Thread Group\n");
		threadGroup.list();

		// Write information about the status of the Thread objects to the console
		Thread[] threads=new Thread[threadGroup.activeCount()];
		threadGroup.enumerate(threads);
		for (int i=0; i<threadGroup.activeCount(); i++) {
			System.out.printf("Thread %s: %s\n",threads[i].getName(),threads[i].getState());
		}

		// Wait for the finalization of the Threadds
		waitFinish(threadGroup);
		
		// Interrupt all the Thread objects assigned to the ThreadGroup
		threadGroup.interrupt();
	}

	/**
	 * Method that waits for the finalization of one of the ten Thread objects
	 * assigned to the ThreadGroup
	 * @param threadGroup
	 */
	private static void waitFinish(ThreadGroup threadGroup) {
		while (threadGroup.activeCount()>9) {
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
```

- Result.java

```java
/**
 * Class that stores the result of the search
 *
 */
public class Result {
	
	/**
	 * Name of the Thread that finish
	 */
	private String name;
	
	/**
	 * Read the name of the Thread
	 * @return The name of the Thread
	 */
	public String getName() {
		return name;
	}
	
	/**
	 * Write the name of the Thread
	 * @param name The name of the Thread
	 */
	public void setName(String name) {
		this.name = name;
	}

}
```

- SearchTask.java

```java
/**
 * Class that simulates a search operation
 *
 */
public class SearchTask implements Runnable {

	/**
	 * Store the name of the Thread if this Thread finish and is not interrupted
	 */
	private Result result;
	
	/**
	 * Constructor of the class
	 * @param result Parameter to initialize the object that stores the results
	 */
	public SearchTask(Result result) {
		this.result=result;
	}

	@Override
	public void run() {
		String name=Thread.currentThread().getName();
		System.out.printf("Thread %s: Start\n",name);
		try {
			doTask();
			result.setName(name);
		} catch (InterruptedException e) {
			System.out.printf("Thread %s: Interrupted\n",name);
			return;
		}
		System.out.printf("Thread %s: End\n",name);
	}
	
	/**
	 * Method that simulates the search operation
	 * @throws InterruptedException Throws this exception if the Thread is interrupted
	 */
	private void doTask() throws InterruptedException {
		Random random=new Random((new Date()).getTime());
		int value=(int)(random.nextDouble()*100);
		System.out.printf("Thread %s: %d\n",Thread.currentThread().getName(),value);
		TimeUnit.SECONDS.sleep(value);
	}

}
```

- 输出结果: ThreadGroup的使用方法


## Processing uncontrolled exceptions in a group of threads

- Main.java

```java
/**
 * Main class of the example
 *
 */
public class Main {

	/**
	 * Main method of the example. Creates a group of threads of
	 * MyThreadGroup class and two threads inside this group
	 * @param args
	 */
	public static void main(String[] args) {

		// Create a MyThreadGroup object
		MyThreadGroup threadGroup=new MyThreadGroup("MyThreadGroup");
		// Create a Taks object
		Task task=new Task();
		// Create and start two Thread objects for this Task
		for (int i=0; i<2; i++){
			Thread t=new Thread(threadGroup,task);
			t.start();
		}
	}

}
```

- MyThreadGroup.java

```java
/**
 * Class that extends the ThreadGroup class to implement
 * a uncaught exceptions method 
 *
 */
public class MyThreadGroup extends ThreadGroup {

	/**
	 * Constructor of the class. Calls the parent class constructor
	 * @param name
	 */
	public MyThreadGroup(String name) {
		super(name);
	}


	/**
	 * Method for process the uncaught exceptions
	 */
	@Override
	public void uncaughtException(Thread t, Throwable e) {
		// Prints the name of the Thread
		System.out.printf("The thread %s has thrown an Exception\n",t.getId());
		// Print the stack trace of the exception
		e.printStackTrace(System.out);
		// Interrupt the rest of the threads of the thread group
		System.out.printf("Terminating the rest of the Threads\n");
		interrupt();
	}
}
```

- Task.java

```java
/**
 * Class that implements the concurrent task
 *
 */
public class Task implements Runnable {

	@Override
	public void run() {
		int result;
		// Create a random number generator
		Random random=new Random(Thread.currentThread().getId());
		while (true) {
			// Generate a random number a calculate 1000 divide by that random number
			result=1000/((int)(random.nextDouble()*1000));
			System.out.printf("%s : %f\n",Thread.currentThread().getId(),result);
			// Check if the Thread has been interrupted
			if (Thread.currentThread().isInterrupted()) {
				System.out.printf("%d : Interrupted\n",Thread.currentThread().getId());
				return;
			}
		}
	}
}
```

- 输出结果: 继承ThreadGroup,来实现uncaughtException方法


## Creating threads through a factory

- Main.java

```java
/**
 * Main class of the example. Creates a Thread factory and creates ten 
 * Thread objects using that Factory 
 *
 */
public class Main {

	/**
	 * Main method of the example. Creates a Thread factory and creates 
	 * ten Thread objects using that Factory
	 * @param args
	 */
	public static void main(String[] args) {
		// Creates the factory
		MyThreadFactory factory=new MyThreadFactory("MyThreadFactory");
		// Creates a task
		Task task=new Task();
		Thread thread;
		
		// Creates and starts ten Thread objects
		System.out.printf("Starting the Threads\n");
		for (int i=0; i<10; i++){
			thread=factory.newThread(task);
			thread.start();
		}
		// Prints the statistics of the ThreadFactory to the console
		System.out.printf("Factory stats:\n");
		System.out.printf("%s\n",factory.getStats());
		
	}

}
```

- MyThreadFactory.java

```java
/**
 * Class that implements the ThreadFactory interface to
 * create a basic thread factory
 *
 */
public class MyThreadFactoryMyThreadFactory.java implements ThreadFactory {

	// Attributes to save the necessary data to the factory
	private int counter;
	private String name;
	private List<String> stats;
	
	/**
	 * Constructor of the class
	 * @param name Base name of the Thread objects created by this Factory
	 */
	public MyThreadFactory(String name){
		counter=0;
		this.name=name;
		stats=new ArrayList<String>();
	}
	
	/**
	 * Method that creates a new Thread object using a Runnable object
	 * @param r: Runnable object to create the new Thread
	 */
	@Override
	public Thread newThread(Runnable r) {
		// Create the new Thread object
		Thread t=new Thread(r,name+"-Thread_"+counter);
		counter++;
		// Actualize the statistics of the factory
		stats.add(String.format("Created thread %d with name %s on %s\n",t.getId(),t.getName(),new Date()));
		return t;
	}
	
	/**
	 * Method that returns the statistics of the ThreadFactory 
	 * @return The statistics of the ThreadFactory
	 */
	public String getStats(){
		StringBuffer buffer=new StringBuffer();
		Iterator<String> it=stats.iterator();
		
		while (it.hasNext()) {
			buffer.append(it.next());
		}
		
		return buffer.toString();
	}

}
```

- Task.java

```java
public class Task implements Runnable {

	@Override
	public void run() {
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}

```

- 输出结果: 继承ThreadFactory类,实现newThread方法.和设计模式中的工厂模式很像,线程也有自己的工厂(易于维护,易于控制数量和易于统计生成的数据),缺点是你得保证应用中所有的线程是通过ThreadFactory创建的.


## 

- Main.java

```java

```

- 

```java

```

- 输出结果: 


