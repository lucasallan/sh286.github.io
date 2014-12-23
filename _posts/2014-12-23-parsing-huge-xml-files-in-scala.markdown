---
layout: post
title:  "Parsing huge xml files in Scala"
date:   2014-12-23 10:49:00
---

I had to parse a [PCap][pcap] XML file (over 100GB uncompressed) and unfortunately most of xml parsers tries to load the whole file in memory - which clearly doesn't work for me.

I built a parser that splits the file in several records and parse them individually.

{% highlight scala %}

object XMLStream {

  val startTag = "<packet>"
  val endTag = "</packet>"

  def apply(filePath: String) = {
    val fileStream: Iterator[String] = Source.fromFile(filePath).getLines
    parse(fileStream)
  }

  private def parse(stream: Iterator[String]) = {
    stream.foldLeft("") { (values, line) =>
      if (line.startsWith(startTag)) {
        line
      }
      else if (line.startsWith(endTag)) {
        storePacket(values + line)
        ""
      } else {
        values + line
      }
    }
  }

  private def parseRecord(packet: String) = {
    val parsed  = scala.xml.XML.loadString(packet)
    // do something
  }
}
{% endhighlight %}


I ran into some performance issues when I was using [tail-recursive][tailrec] in the `parse` method, so I had to switch to `foldLeft` and now everything works just fine.

[tailrec]: http://www.scala-lang.org/api/current/index.html#scala.annotation.tailrec
[pcap]: http://en.wikipedia.org/wiki/Pcap

