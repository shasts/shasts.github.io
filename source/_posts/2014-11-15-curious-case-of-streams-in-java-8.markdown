---
layout: post
title: "Curious case of streams in Java 8"
comments: true
categories: 
---
<p>I have been experimenting with Java 8 steams API and thinking about using it in projects I work on. Found the API to create a <code> parallel stream</code> from a <code>list</code> attractive utility for some computations.</p>
<p>I have been also thinking of the overhead of a <code>parallel stream</code> for not so large datasets.</p>
<p> One thing that intrigued me was the ability to get a <code>parallel stream</code> from a <code>spliterator</code>, which is available on available on all <code>Iterable</code>.</p>
<p>Did some experiments with parallel streams and noticed some real performance loss for some <code>Iterable</code>. 
The reason is, <code>spliterator</code> decides how fast the parallel stream can be. One of the job <code>spliterator</code> has to do is to decompose data into two parts, which can be processed parallel.</p>
<p>For example, in case of a array backed list, <code>spliterator</code> can determine how data can be decomposed in to two parts finely before hand. However, in case of linked list, spliterator can never be precise about the length of the data set. So it splits the data into first and rest, which is as bad as it can get.</p>

Below is the code used for benchmarking using <a href="http://openjdk.java.net/projects/code-tools/jmh/">JMH</a>.

The test class
{% codeblock lang:java%}
public class ParallelStreamSupportTest {

    @Benchmark
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.MILLISECONDS)
    public void intStream(IntStreamProvider provider) {
        provider.getStream().mapToInt(i -> i + 2).sum();
    }

    @Benchmark
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.MILLISECONDS)
    public void streamFromContiguousSet(ContiguousStreamProvider provider) {
        provider.getStream().mapToInt(i -> i + 2).sum();
    }

    @Benchmark
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.MILLISECONDS)
    public void listIterator(StreamOverListProvider provider) {
        provider.getStream().mapToInt(i -> i + 2).sum();
    }

}
{% endcodeblock %}

The provider interface.

{% codeblock lang:java%}
public interface StreamProvider {

    final static int SIZE = 10000000;

    public Stream<Integer> getStream();

}
{% endcodeblock %}

Provider for <code>IntStream</code>

{% codeblock lang:java%}
@State(Scope.Thread)
public class IntStreamProvider implements StreamProvider {

    public Stream<Integer> getStream() {
        return IntStream.rangeClosed(1, SIZE).boxed().parallel();
    }
}
{% endcodeblock %}

Provider for a stream from <code>ArrayList</code>

{% codeblock lang:java%}
@State(Scope.Thread)
public class StreamOverListProvider implements StreamProvider {

    private List<Integer> integers;

    public StreamOverListProvider() {
        Set<Integer> integer = ContiguousSet.create(Range.closed(1, SIZE), DiscreteDomain.integers());
        integers = new ArrayList<>();
        integers.addAll(integer);
    }

    public Stream<Integer> getStream() {
        return integers.parallelStream();
    }
}
{% endcodeblock %}

Provider for a stream from a <code>Set</code>

{% codeblock lang:java%}
@State(Scope.Thread)
public class ContiguousStreamProvider implements StreamProvider {

    private Set<Integer> integers;

    public ContiguousStreamProvider() {
        integers = ContiguousSet.create(Range.closed(1, SIZE), DiscreteDomain.integers());
    }

    public Stream<Integer> getStream() {
        return StreamSupport.stream(integers.spliterator(), true);
    }
}
{% endcodeblock %}

<p>Below is the benchmark result.</p>
{% codeblock lang:haskell%}
Benchmark                                                Mode  Samples   Score   Error  Units
o.s.ParallelStreamSupportTest.intStream                  avgt      200  26.438 ± 0.404  ms/op
o.s.ParallelStreamSupportTest.listIterator               avgt      200  15.549 ± 0.355  ms/op
o.s.ParallelStreamSupportTest.streamFromContiguousSet    avgt      200  56.330 ± 0.309  ms/op
{% endcodeblock %}

<p>From the result it quite visible that a <code>ParallelStream</code> over an ArrayList is the fastest one, with 15.549 ms/op. However, the benchmarking code has a flaw, where the collection behind <code>IntStream</code> is constructed for each iteration, where as the actual collection for other streams are pre-constructed and only stream is constructed for each iteration. But since that is not the purpose of benchmarking, I decided to ignore that, as only relative performance need to be known. </p>
<p>So one could consider <code>IntStream</code> and <code>Stream</code> over an <code>ArrayList</code> being equally fast. But stream over a <code>Set</code> is much slower compared to others.</p>