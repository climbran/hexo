title: Iterator的实现
date: 2015-08-1 17:26:08
tags: [设计模式,java]
---
<pre>
<code>
interface Iterator{
	boolean end();
	Object current();
	void next();
}

public class Sequence{
	private Object[] items;
	private next = 0;
	
	public Sequence(int size){
		items = new Objec[size];
	}
	
	public void add(Object x){
		if(next&ltitems.length)
			items[next++] = x;
	}
	
	private class SequenceIterator implements Iterator{
		private int i = 0;
		public boolean end(){
			return i == items.length;
		}
		public Object current(){
			return items[i];
		}
		public void next(){
			if(i&ltitems.length)
				i++;
		}
	}
	
	pubic Iterator iterator(){
		return new SequenceIterator();
	}
}
</code>
</pre>