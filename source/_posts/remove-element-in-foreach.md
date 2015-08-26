title: 在foreach循环中移除元素的方法
date: 2015-06-18 11:18:18
tags: [java]
---
foreach循环中无法直接对Iterable中的元素进行remove，可以通过下面方式实现:
<pre>
<code><xmp>for(String s : new ArrayList<String>().addAll(list)){ //list为一个List<String>
	if(s.equals("need remove"))
		list.remove(s);
}
</xmp></code></pre>