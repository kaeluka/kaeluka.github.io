---
layout: post
title:  "Tracing as a Service"
date:   2016-10-29 11:41:30 +0200 
categories: work
permalink: spencer
---

<div style="width: 35%; display: block; margin: 0 auto;">
  <img src="http://stbr.me/assets/spencer_logo_bright.svg" alt="Spencer logo"/>
</div>

<br/>A project I've been working on is called Spencer. Spencer is a set of tools
with the aim to make it trivially easy for language designers, runtime
developers and other language technologists to understand the behaviour of
"typical" programs on the JVM.

Spencer contains a tool that works as a drop in replacement for `java` -- it
starts and runs an application, but recompiles code dynamically to log
information about variable loads/stores, field loads/stores, and method
enters/exits. These events are tagged with the current thread id, a sequence
number, types of callees, and so on.

We are going to host these data as a web service for people to interact with.
Users will be able to query the web service by constructing, interactively,
queries that select objects. These queries can be gradually refined using an
accessible interface, and visualised in interactive diagrams.

Since data sets -- once they are uploaded -- never change, we can apply caching
to these queries, which will lead to the ability to analyse terabytes of data
interactively.

# Download
I presented an high level overview at an
[UpScale](https://upscale.project.cwi.nl/) meeting in Oslo a few weeks back. You
can find the slides below.


 - [Slides (pdf)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_slides.pdf?raw=true)
 - [Slides (keynote)](https://github.com/kaeluka/kaeluka.github.io/blob/master/assets/spencer_slides.key?raw=true)

# Technical Information

<div>
        <h1>Spencer, Tracing as a Service</h1>

        <p>
            Spencer is a tool to analyse large program traces using custom queries.
            Queries in spencer are composable: If you have one analysis, you are
            able to refine it using others.
        </p>

        <h2>Queries</h2>

        <p>
            Queries in Spencer are expressions that return a set of object IDs. They
            implement a test: does an object's usage fulfill a certain definition or
            not?
        </p>

        <p>
            Spencer distinguishes between primitive queries and composed queries.
            Primitive queries are the basic building blocks that are implemented
            "natively" in the backend of the service.
        </p>

        <p>
            The primitive queries are:
        </p>

        <style>
            table thead tr td {
                background-color: #ebdaa9;
            }

            table {
                margin-top: 2em;
                margin-bottom: 2em;
                margin-left: auto;
                margin-right: auto;
            }
        </style>

        <script>
            var COMMAND='perobj';
            var DBNAME='test';
        </script>

        <table class="queries">
            <thead>
                <tr>
                    <td>Query</td>
                    <td>Meaning</td>
                </tr>
            </thead>
            <tbody>
            
                <tr>
                    <td><div class="query">MutableObj()</div></td>
                    <td><span class="queryExplanation">All objects that are changed outside of their constructor.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">ImmutableObj()</div></td>
                    <td><span class="queryExplanation">All objects that are never changed outside their constructor.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">StationaryObj()</div></td>
                    <td><span class="queryExplanation">All objects that are never changed after being read from for the first time.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">UniqueObj()</div></td>
                    <td><span class="queryExplanation">All objects that have at most one active reference at each time.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">HeapUniqueObj()</div></td>
                    <td><span class="queryExplanation">All objects that have at most one active heap reference at each time.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">TinyObj()</div></td>
                    <td><span class="queryExplanation">All objects that do not have or use reference type fields.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">StackBoundObj()</div></td>
                    <td><span class="queryExplanation">All objects that never escape to the heap.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">InstanceOfClass(java.lang.String)</div></td>
                    <td><span class="queryExplanation">All objects that are instances of class java.lang.String.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">AllocatedAt(somefile:123)</div></td>
                    <td><span class="queryExplanation">All objects that were allocated at somefile:123.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">Obj()</div></td>
                    <td><span class="queryExplanation">All objects that were traced.</span></td>
                </tr>
            
            </tbody>
        </table>

        <p>
            Spencer's power lies in the fact that these queries all return the same
            kind of structure: sets of objects! This restriction makes it possible
            to compose queries into larger ones, like so:
        </p>

        <table class="queries">
            <thead>
                <tr>
                    <td>Query</td>
                    <td>Meaning</td>
                </tr>
            </thead>
            <tbody>
            
                <tr>
                    <td><div class="query">And(ImmutableObj() AllocatedAt(String.java:1933))</div></td>
                    <td><span class="queryExplanation">All objects that are never changed outside their constructor, and that were allocated at String.java:1933.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">Or(UniqueObj() ImmutableObj())</div></td>
                    <td><span class="queryExplanation">All objects that have at most one active reference at each time, and that are never changed outside their constructor.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">Deeply(Or(UniqueObj() ImmutableObj()))</div></td>
                    <td><span class="queryExplanation">All objects that an explanation of Deeply(Or(UniqueObj() ImmutableObj())).</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">ReachableFrom(AllocatedAt(String.java:1933))</div></td>
                    <td><span class="queryExplanation">All objects that are reachable from any objects that were allocated at String.java:1933.</span></td>
                </tr>
            
                <tr>
                    <td><div class="query">CanReach(AllocatedAt(String.java:1933))</div></td>
                    <td><span class="queryExplanation">All objects that can reach any objects that were allocated at String.java:1933.</span></td>
                </tr>
            
            </tbody>
        </table>

        <h2>What's in the data?</h2>

        <p>
            Spencer contains many data sets. One data set is a program trace (in preprocessed form) that
            stems from running and instrumenting a program.
            The data in the program are distributed over several tables.
            Although these tables are not accessed directly by a user (they are
            accessed by the primitive queries instead), a user that wants to help
            Spencer grow must understand the data that they contain.
        </p>

        <h3>The <span class="code">objects</span> table</h3>

        <p>
            The objects table contains basic information on every object that was
            encountered during tracing. But see for yourself, below are a few
            records from our database. As you can see, we identify each object by
            its unique numeric ID, an object has an optional allocation site, it
            was created at a certain event ID, and we also record the last time the
            object was used (loaded, modified, read from, or called).
        </p>
        <p>
            You notice that there's one line with a negative ID. Those
            "objects" are pseudo objects that represent a class. Every access to
            static fields and methods will be reported as an access to this pseudo
            object in the rest of the data.
        </p>

        <p>
            You might also notice that the allocation sites are optional.
            This is a technical limitation: First, some class files do not contain
            location information and there is no line we could give. Second,
            some objects are created very early in the startup process, before
            the instrumentation is running. Third, some objects are allocated by
            native code -- and we do not instrument native code.
        </p>

        <table>
            <thead>
                <tr>
                    <td>object id</td>
                    <td>allocation site?</td>
                    <td>event idx of allocation </td>
                    <td>event idx of last usage</td>
                </tr>
            </thead>
            <tbody>
                
                    <tr>
                        <td><span class="oid">9601</span></td>
                        <td>
                            <span class='empty'>none</span>
                        </td>
                        <td><span class="evtidx">402171</span></td>
                        <td><span class="evtidx">402362</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">13719</span></td>
                        <td>
                            <span class='allocationsite'>String.java:1933</span>
                        </td>
                        <td><span class="evtidx">808782</span></td>
                        <td><span class="evtidx">808797</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">21992</span></td>
                        <td>
                            <span class='allocationsite'>StringBuilder.java:407</span>
                        </td>
                        <td><span class="evtidx">1714178</span></td>
                        <td><span class="evtidx">1714547</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">22233</span></td>
                        <td>
                            <span class='allocationsite'>String.java:207</span>
                        </td>
                        <td><span class="evtidx">1743966</span></td>
                        <td><span class="evtidx">1743989</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">25218</span></td>
                        <td>
                            <span class='allocationsite'>ConcurrentHashMap.java:2450</span>
                        </td>
                        <td><span class="evtidx">1980872</span></td>
                        <td><span class="evtidx">1987815</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">31798</span></td>
                        <td>
                            <span class='allocationsite'>String.java:207</span>
                        </td>
                        <td><span class="evtidx">2795792</span></td>
                        <td><span class="evtidx">2795837</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">41337</span></td>
                        <td>
                            <span class='allocationsite'>String.java:207</span>
                        </td>
                        <td><span class="evtidx">4125910</span></td>
                        <td><span class="evtidx">4126699</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">68624</span></td>
                        <td>
                            <span class='allocationsite'>ZipFile.java:310</span>
                        </td>
                        <td><span class="evtidx">7736050</span></td>
                        <td><span class="evtidx">7736294</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">88484</span></td>
                        <td>
                            <span class='allocationsite'>ZipFile.java:557</span>
                        </td>
                        <td><span class="evtidx">11927015</span></td>
                        <td><span class="evtidx">11927112</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">99233</span></td>
                        <td>
                            <span class='allocationsite'>String.java:1969</span>
                        </td>
                        <td><span class="evtidx">13004953</span></td>
                        <td><span class="evtidx">13005348</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">103860</span></td>
                        <td>
                            <span class='allocationsite'>ZipCoder.java:89</span>
                        </td>
                        <td><span class="evtidx">13423753</span></td>
                        <td><span class="evtidx">13423761</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">-106632</span></td>
                        <td>
                            <span class='empty'>none</span>
                        </td>
                        <td><span class="evtidx">13663550</span></td>
                        <td><span class="evtidx">13668848</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">107795</span></td>
                        <td>
                            <span class='allocationsite'>String.java:1933</span>
                        </td>
                        <td><span class="evtidx">13757898</span></td>
                        <td><span class="evtidx">13758322</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td>
                            <span class='allocationsite'>String.java:2647</span>
                        </td>
                        <td><span class="evtidx">18072956</span></td>
                        <td><span class="evtidx">18073027</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">149948</span></td>
                        <td>
                            <span class='allocationsite'>String.java:491</span>
                        </td>
                        <td><span class="evtidx">18303524</span></td>
                        <td><span class="evtidx">18303738</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">158519</span></td>
                        <td>
                            <span class='allocationsite'>Benchmark.java:643</span>
                        </td>
                        <td><span class="evtidx">19237256</span></td>
                        <td><span class="evtidx">19264472</span></td>
                    </tr>
                
                <tr>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                </tr>
            </tbody>

        </table>

        <h3>The <span class="code">refs</span> table</h3>

        <p>
            The <span class="code">refs</span> table contains the "graph
            structure" of a program trace.
        </p>

        <p>
            References in this table are either variable- or field-references.
            References have a caller (the object holding the reference), a
            callee (the object being referenced, or <span class="oid">0</span>),
            they have a name (local variable names are not recorded, just
            numbered like <span class="code">var_{i}</span>).
        </p>

        <p>
            References also have a time of when they where established (by
            setting a field or variable, or passing a method argument) and a
            time when they where deleted (when overwriting a field or variable,
            <span class="todo">for local variables when returning from a method
                call or for fields when an object is not used any
                longer</span>).
        </p>

        <table>
            <thead>
                <tr>
                    <td><span class="code">caller</span></td>
                    <td class="empty"></td>
                    <td><span class="code">kind</span></td>
                    <td class="empty"></td>
                    <td><span class="code">name</span></td>
                    <td class="empty"></td>
                    <td><span class="code">callee</span></td>
                    <td class="empty"></td>
                    <td><span class="code">start</span></td>
                    <td class="empty"></td>
                    <td><span class="code">end</span>?</td>
                    <td class="empty"></td>
                    <td><span class="code">thread</span></td>
                </tr>
            </thead>
            <tbody>
                
                    <tr>
                        <td><span class="oid">25218</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">field</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">key</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">25168</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">1980877</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">25218</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">field</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">val</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">25017</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">1980879</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">25218</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">field</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">next</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">0</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">1980881</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">107795</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">field</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">value</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">107796</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">13757912</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">PmdThread 1</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">var</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">var_1</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">146965</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">18072957</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">field</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">value</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">146967</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">18072970</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">var</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">var_1</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">146593</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">18072974</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='evtidx'>18072977</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">var</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">var_1</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">146593</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">18072977</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">var</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">var_3</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">146967</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">18072979</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>has</emph></td>
                        <td><span class="code">var</span></td>
                        <td style="padding: 0 0.5em;"><emph>called</emph></td>
                        <td><span class="code">var_5</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="oid">146594</span></td>
                        <td style="padding: 0 0.5em;"><emph>from</emph></td>
                        <td><span class="evtidx">18072982</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class='empty'>none</span></td>
                        <td style="padding: 0 0.5em;"><emph>established by thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                <tr>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                </tr>
            </tbody>

        </table>

        <h3>The <span class="code">uses</span> table</h3>

        <p>
            Accesses to fields of objects end up in the
            <span class="code">uses</span> table. They are represented by events
                of kind <span class="code useKind">read</span> or
                <span class="code useKind">modify</span> for reads or writes of
                primitive type fields and
                <span class="code useKind">fieldload</span> or
                <span class="todo code useKind">fieldstore</span> for reads or writes of
                reference type fields.
        </p>

        <table>
            <thead>
                <tr>
                    <td><span class="code">caller</span></td>
                    <td class="empty"></td>
                    <td><span class="code">kind</span></td>
                    <td class="empty"></td>
                    <td><span class="code">name</span></td>
                    <td class="empty"></td>
                    <td><span class="code">callee</span></td>
                    <td class="empty"></td>
                    <td><span class="code">idx</span></td>
                    <td class="empty"></td>
                    <td><span class="code">thread</span></td>
                </tr>
            </thead>
            <tbody>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">modify</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276301</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">read</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276333</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">modify</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276334</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">fieldload</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">data</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276335</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">fieldload</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">data</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276336</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">read</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276337</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">read</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276344</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">modify</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276345</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">read</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276348</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">read</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">pos</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">73962</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8276355</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">67863</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">fieldload</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">value</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">67863</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8389976</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">67863</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">fieldload</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">value</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">67863</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8389987</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">67863</span></td>
                        <td style="padding: 0 0.5em;"><emph>executes</emph></td>
                        <td><span class="code useKind">fieldload</span></td>
                        <td style="padding: 0 0.5em;"><emph>of</emph></td>
                        <td><span class="code">value</span></td>
                        <td style="padding: 0 0.5em;"><emph>of obj</emph></td>
                        <td><span class="oid">67863</span></td>
                        <td style="padding: 0 0.5em;"><emph>at event</emph></td>
                        <td><span class="evtidx">8389988</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                <tr>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                </tr>
            </tbody>

        </table>

        <h3>The <span class="code">calls</span> table</h3>

        <p>
            Calls on objects are stored in the calls table. The calls table
            contains timing and thread information. Since calls are tagged with
            the event index of their start and end, we know that if two calls
            in the same thread are nested temporarily, they must also be nested
            on the call stack. The "apropos" view, accessible from object links
            exploits this for visulisation.
        </p>

        <p><emph>Hint:</emph> try brushing along the event indices in the
            <span class="code">start</span> column. The pattern in which the
        events in the <span class="code">end</span> column turn red tells you
        something about the call stack! Try to figure it out as an exercise :)
        </p>

        <table>
            <thead>
                <tr>
                    <td><span class="code">caller</span></td>
                    <td class="empty"></td>
                    <td><span class="code">name</span></td>
                    <td class="empty"></td>
                    <td><span class="code">callee</span></td>
                    <td class="empty"></td>
                    <td><span class="code">callee</span></td>
                    <td class="empty"></td>
                    <td><span class="code">start</span></td>
                    <td class="empty"></td>
                    <td><span class="code">end</span></td>
                    <td class="empty"></td>
                    <td><span class="code">thread</span></td>
                </tr>
            </thead>
            <tbody>
                
                    <tr>
                        <td><span class="oid">-21444</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">copyOfRange</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">-21444</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">String.java:207</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">1714182</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">1714193</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">21992</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">indexOf</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">21992</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">String.java:1503</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">1714287</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">1714411</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">21992</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">lastIndexOf</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">21992</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">String.java:1611</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">1714419</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">1714545</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">-20845</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">min</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">-20845</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">String.java:1653</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">1714423</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">1714424</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">-21797</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">requireNonNull</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">-21797</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">ZipEntry.java:118</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">2313150</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">2313155</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">32347</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">hashCode</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">32347</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">CodeSource.java:120</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">5642604</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">5642607</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">32347</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">equals</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">32347</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">CodeSource.java:153</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">5642638</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">5642796</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">54314</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">matchCerts</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">54314</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">CodeSource.java:157</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">5642798</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">5642806</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">-21444</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">copyOfRange</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">-21444</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">String.java:207</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">18072960</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">18072969</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                    <tr>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>calls method</emph></td>
                        <td><span class="code useKind">startsWith</span></td>
                        <td style="padding: 0 0.5em;"><emph>on object</emph></td>
                        <td><span class="oid">146966</span></td>
                        <td style="padding: 0 0.5em;"><emph>at callsite</emph></td>
                        <td><span class="sourceLocation">String.java:1434</span></td>
                        <td style="padding: 0 0.5em;"><emph>from event</emph></td>
                        <td><span class="evtidx">18072976</span></td>
                        <td style="padding: 0 0.5em;"><emph>to</emph></td>
                        <td><span class="evtidx">18073026</span></td>
                        <td style="padding: 0 0.5em;"><emph>in thread</emph></td>
                        <td><span class="code">main</span></td>
                    </tr>
                
                <tr>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                    <td></td>
                    <td>...</td>
                </tr>
            </tbody>

        </table>

        <h3>Future Work: The <span class="code">sync</span> table</h3>

        <p>
            Event indices are numeric. This wrongly suggests that events are ordered
            by index, even accross threads. This is wrong because the fact that two
            parallel events ended up in the data base in one particular order does
            not imply that there was an actual happens-before relation.
        </p>
        <p>
            Spencer might, in the future, trace synchronisation points between
            threads that establishes happens-before accross thread boundaries.
        </p>

        <table>
            <thead>
                <tr>
                    <td><span class="code">sync1-thread</span></td>
                    <td><span class="code">sync1-file</span></td>
                    <td><span class="code">sync1-line</span></td>
                    <td><span class="code">sync1-event-before</span></td>
                    <td><span class="code">sync2-thread</span></td>
                    <td><span class="code">sync2-file</span></td>
                    <td><span class="code">sync2-line</span></td>
                    <td><span class="code">sync2-event-after</span></td>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                    <td>...</td>
                </tr>
            </tbody>
        </table>

</div>
