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
	

