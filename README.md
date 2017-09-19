# Chapter 2 Thread Safety

`It's far easier to design a class to be thread-safe than to retrofit it for thread safety later`

## race condition

ex) stateful servlet: having counter at class field and is incremented at #service()

ex2) lazy initialization: #getInstance() w/o thread-safe handling

	public X getInstance() {
		if (instance == null) {
			instance = new X();
		}
		return instance;
	}

ex3) violation of atomicity: paired objects can't be separated each other which should be atomic

	private final AtomicReference<> lastNumber = ...
	private final AtomicReference<> lastFactors = ...
	public void service() {
		i = extractFromRequest();
		if (i equals lastNumber#get()) {
			encodeIntoResponse(lastFactors#get());
			// do something
		} else {
			setLastNumber(x);
			lastFactors#set(x);
			// do something
		}
	}

## reentrancy

same lock owner can enter the synchronized block more than once (how many the owner enter the block is counted)

Java's `synchronized` is reentrant

## Guarding state with locks

why Vector (thread safe) class is no better than ArrayList?

	if (!vector#contains(element)) vector#add(element);

above code is NOT thread safe even though #contains() and #add() are thread-safe methods

# Chapter 3 Sharing objects

## Visibility

don't change the states (variables) w/o synchronization if the states are referred in threads.

	... SomeClass ext. Thread {
		public void run() {
			while (!ready)
				Thread.yield();
			Sys.out.pln(number);
		}

		... main() {
			new ReaderThread().start();
			number = 42;	// MUST BE IN synch block
			ready = true; // MUST BE IN synch block
		}

### stale data

		class NotThreadSafeInteger {
			private int val;
			pub. int get() { return val;}
			pub. void set(int val) { this.val = val;}
		}

	(!) `double` `long` is not one-action operation in 32bit JVM. Be careful!

### Volatile variables

*Locking can guarantee both visibility and atomicity; volatile variables can only guarantee visibility*

	volatile boolean asleep;
	...
	while (!asleep)
		countSomeSheep();

above is a good example for use of volatile variables.

*However, if `volatile` keyword is omitted, it won't work on deployment env even if it works on development env. The reason is that some server container optimizes code so that `asleep` be considered as constant value; server doesn't know other thread can change the variable... To avoid such, use `-server` JVM command line option so that such behavior can appear on development env as well*

bad example of volatile variables is to use the variables as incremented counters.

When to use volatile variables are:

- Writes to the variable don't depend on its current value (counter), or you can ensure that only a single thread ever updates the value
- The variable doesn't participate in invariants w/ other state variables
- Locking is not required for any other reason while he variable is being accessed

## Publication and escape

	pub. static Set<Secret> knownSecrets;
	pub. void init() { knownSecrets = new HashSet<Secret>();}

above is a typical example of escaping by publish. Both knownSecrets and Secret are exposed to public. What is worse, adding new Secret also get revealed to public.

	public ThisEscape(EventSource src) {
		src.registerListener(
			new EventListener() {
				public void onEvent(Event e) { doSomething(e);}
			});
	}

'Event e' makes 'this' reference escape since 'e' contains a hidden reference to the enclosing instance.
Publishing an object from within its constructor is incomplete state even if the publication is the last statement in the constructor.

*A common mistake that can let the 'this' reference escape during construction is to start a thread from a constructor. When an object creates a thread from its constructor, it almost always shares its 'this' reference with the new thread. There's nothing wrong with creating a thread in a constructor, but it's best not to start the thread immediately. Instead, expose a 'start' or 'initialize' method that starts the owned thread*

Code below shows factory method to prevent the 'this' reference from escaping during construction.

	public class SafeListener {
		private final EventListener listener;
		private SafeListener() {
			listener = new EventListener() {
				public void onEvent(Event e) { doSomething(e);}
			}
		};
	}
	// factory class
	public static SafeListener newInstance(EventSource src) {
		SafeListener safe = new SafeListener();
		// now constructor is done and safe object is fully ready
		src.registerListener(safe.listener);
		return safe;
	}

## Thread confinement

* no shared variables (use local variables within a thread)
* use of pooled JDBC 'Connection' object

### Ad-hoc thread confinement

### Stack confinement

* local variables in a method are thread-safe unless it escapes by appending to parameter or set as returned value

### ThreadLocal

* 'ThreadLocal<T>' is like 'Map<Thread, T>' each thread has its own 'T'

## Immutability
