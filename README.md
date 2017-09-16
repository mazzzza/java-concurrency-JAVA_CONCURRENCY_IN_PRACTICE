# Chapter2 Thread Safety

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



	
