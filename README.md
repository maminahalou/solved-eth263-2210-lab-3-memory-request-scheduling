Download Link: https://assignmentchef.com/product/solved-eth263-2210-lab-3-memory-request-scheduling
<br>
<h1>1. Introduction</h1>

In this lab, you will implement and evaluate two memory scheduling policies: ATLAS [1] and BLISS [2]. To this end, you will extend Ramulator [3], which is a publicly-available architectural memory simulator, to model the two memory scheduling policies. Ramulator implements many DRAM standards (e.g., DDR4, LPDDR4, WideIO). It also implements a simple out-of-order processor frontend that enables Ramulator to simulate workloads as a standalone tool. We provide more information on Ramulator and guide you through how to use it in Section 2.

The version of Ramulator that we provide you with already models memory scheduling policies such as first-come-first-serve (FCFS) and first-ready-first-come-first-serve (FRFCFS). In this assignment, your job is to implement the ATLAS and BLISS scheduling policies as specified later in this handout.

<h1>2. A Short Ramulator Tutorial</h1>

Please follow this short tutorial to learn how to use Ramulator. Here is the outline of the tutorial:

<ul>

 <li>Downloading and Building Ramulator</li>

 <li>Simulation Modes</li>

 <li>Simulator Output</li>

 <li>Configuration Files</li>

 <li>Simulating multi-programmed workloads</li>

 <li>Code Organization</li>

</ul>

<h2>2.1. Downloading and Building Ramulator</h2>

Ramulator [3] is a fast and cycle-accurate DRAM simulator that supports a wide array of commercial, as well as academic, DRAM standards. Please see the README after downloading Ramulator using the link below.

Exercise 0/6: In Lab 3 on Moodle, we provide you a version of Ramulator that includes memory traces that you will use in this assignment. Please download and extract the tarball:

<a href="https://moodle-app2.let.ethz.ch/pluginfile.php/994834/mod_assign/introattachment/0/ramulator_source.zip?forcedownload=1">https://moodle-app2.let.ethz.ch/pluginfile.php/994834/mod</a> <a href="https://moodle-app2.let.ethz.ch/pluginfile.php/994834/mod_assign/introattachment/0/ramulator_source.zip?forcedownload=1">assign/introattachment/0/ </a><a href="https://moodle-app2.let.ethz.ch/pluginfile.php/994834/mod_assign/introattachment/0/ramulator_source.zip?forcedownload=1">ramulator</a> <a href="https://moodle-app2.let.ethz.ch/pluginfile.php/994834/mod_assign/introattachment/0/ramulator_source.zip?forcedownload=1">source.zip?forcedownload=1</a>

Ramulator requires a <em><sub>C++11 </sub></em>compiler (e.g., <em><sub>clang++</sub></em>, <em><sub>g++-5</sub></em>). By default, Ramulator will build using <em><sub>clang++</sub></em>. However, you can modify the line starting with CXX:= in <em><sub>‘Makefile’ </sub></em>in the Ramulator directory to pick a different compiler.

Exercise 1/6 Compiling Ramulator

You can easily build Ramulator by running <em><sub>Make</sub></em>:

$ make -j

After successful compilation, you will find a binary executable file called <em><sub>ramulator </sub></em>in the same directory with the Makefile. You can run the binary to print the help message to the terminal:

Exercise 2/6 Printing Ramulator Help Message

$ ./ramulator

Usage: ./ramulator &lt;configs-file&gt; –mode=cpu,dram [–stats &lt;filename&gt;]

&lt;trace-filename1&gt; &lt;trace-filename2&gt;

Example: ./ramulator ramulator-configs.cfg –mode=cpu cpu.trace cpu.trace

<h2>2.2. Simulation Modes</h2>

Ramulator supports three different usage modes:

<ul>

 <li><sub>Memory Trace Driven: </sub>In this mode, Ramulator is provided with an input trace file that contains <em>main memory requests </em>of an application to simulate. Ramulator sequentially processes these requests based on the selected DRAM standard (e.g., DDR4). This mode does not model any system in sufficient detail to perform timing simulations. Because of that, Memory Trace Driven mode is better suited for testing the functionality of newly added features. In this assignment, we will <em><sub>not </sub></em>use this mode. Still, you can find more information about this mode in the public Ramulator repository [4] in case you are interested.</li>

 <li><sub>Gem5 Driven: </sub>Gem5 [5] is a full-system simulator that models CPU architecture in detail. Ramulator can be attached to <em><sub>gem5 </sub></em>to simulate the main memory component of the system. In this assignment, will <em><sub>not </sub></em>use this mode. If interested, you can take a look at <a href="http://gem5.org/">gem5’s homepage</a> for more information about this simulator. You can also find out how to attach Ramulator to gem5</li>

 <li><sub>CPU Trace Driven: </sub>This is the simulation mode that you will need to use in this lab assignment. In this mode, Ramulator directly reads CPU instruction traces from a file, and simulates a simplified outof-order CPU core model that generates memory requests to the DRAM subsystem. Such trace files contain non-memory instructions and memory requests. Depending on how the trace file is generated, the memory requests in the trace file may correspond to a certain cache level or directly to the main memory. If the trace contains main memory requests, we call it a <em>cache-filtered trace</em>. Whensimulating</li>

</ul>

a cache-filtered trace, Ramulator should be configured to not instantiate any caches. Each line in the CPU trace file represents a memory request, and can have one of the following three formats:

<table width="84">

 <tbody>

  <tr>

   <td width="26">hnum</td>

   <td width="6">–</td>

   <td width="52">cpuinsti</td>

  </tr>

 </tbody>

</table>

<table width="71">

 <tbody>

  <tr>

   <td width="32">haddr</td>

   <td width="6">–</td>

   <td width="32">readi</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>hnum-cpuinstihaddr-readi: If a line contains two tokens, the first tokenrepresents the number of CPU (i.e., non-memory) instructions that precede a read request. The second tokenspecifies the memory address of the read request.</li>

</ul>

<table width="104">

 <tbody>

  <tr>

   <td width="32">haddr</td>

   <td width="6">–</td>

   <td width="66">writebacki</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>hnum-cpuinstihaddr-readihaddr-writebacki: The first two tokens in a line with three tokens are the same as in the first format. The third tokenis the decimal address of the writeback request, which is the dirty cache-line eviction caused by the read request before it.</li>

</ul>

<table width="26">

 <tbody>

  <tr>

   <td width="26">hnum</td>

  </tr>

 </tbody>

</table>

<table width="71">

 <tbody>

  <tr>

   <td width="32">haddr</td>

   <td width="6">–</td>

   <td width="32">readi</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>hunusedihunusedihnum-cpuinstihtypeihaddri: hunusedi tokens are used to pass additional information to the simulation, which are not relevant to this assignment and ignored by Ramulator.-cpuinst<sub>i </sub>represents the number of CPU (i.e., non-memory) instructions that precede a memory request. <sub>h</sub>type<sub>i </sub>indicates the type of the memory request, which can be a load (<em><sub>L</sub></em>) request or a store (<em><sub>S</sub></em>) request.specifies the memory address of the request.</li>

</ul>

<table width="585">

 <tbody>

  <tr>

   <td width="585">Exercise 3/6 Running Ramulator in CPU Trace Driven mode:$ ./ramulator configs/<strong>test</strong>-config.cfg –mode=cpu cpu.tracetracenum: 1 trace_list[0]: cpu.traceWarmup <strong>complete</strong>! Resetting stats… Starting the simulation…[0]retired: 26, clk, 224Simulation <strong>done</strong>. Statistics written to DDR4.stats</td>

  </tr>

 </tbody>

</table>

<table width="59">

 <tbody>

  <tr>

   <td width="59">mode=cpu</td>

  </tr>

 </tbody>

</table>

In the above command, <em><sub>‘configs/DDR4-config.cfg’ </sub></em>specifies a Ramulator configuration file that contains several parameters related to the architecture to simulate. The second argument, –, tells Ramulator to run in CPU Trace Driven mode. The last argument, <em><sub>‘cpu.trace’</sub></em>, specifies the path to the input trace file to simulate. <em>‘cpu.trace’ </em>is a very short trace that do not represent any real application but rather is used to demonstrate how to run Ramulator. 2.3. Simulator Output

<table width="316">

 <tbody>

  <tr>

   <td width="42">stats</td>

   <td width="67">hfilenamei</td>

   <td width="175">–</td>

   <td width="32">-mode</td>

  </tr>

 </tbody>

</table>

Ramulator reports a series of statistics for every run. These statistics are written to a file. By default, the filename will be <em>‘</em>h<em>standard name</em>i<em>.stats’ </em>(e.g., <em>‘DDR4.stats’</em>). You can output the statistics using a custom filename by adding — to the command line after the argument.

<table width="585">

 <tbody>

  <tr>

   <td width="585">Exercise 4/6: Understanding the stats file:The simulation in the previous task should have created the stats file <em><sub>‘DDR4.stats’</sub></em>. Open the stats file with your favorite text editor (e.g., <em><sub>emacs</sub></em>, <em><sub>gedit</sub></em>, <em><sub>vim</sub></em>, <em><sub>kate</sub></em>) and find out which statistics Ramulator reports by default. Fill in the values for the metrics below:<em>Note: you do not have to submit the stats that you filled in above. This is just an exercise to help you get familiar with Ramulator’s stats files.</em>1.   <em>Executed Instructions:</em>2.   <em>CPU Cycles:</em>3.   <em>IPC:</em>4.   <em>Row Misses:</em>5.   <em>Row Hits:</em>6.   <em>Row Conflicts:</em>7.   <em>Average access latency:</em>8.   <em>Read Bandwidth:</em>9.   <em>Write Bandwidth:</em></td>

  </tr>

 </tbody>

</table>

<h2>2.4. Configuration Files</h2>

Ramulator enables cycle-accurate simulation of a diverse set of memory technologies. It uses various configuration files to simulate different memory technologies. Now, let’s analyze important parameters in the configuration files.

The pre-defined configuration files are available in the <em><sub>‘configs/’ </sub></em>directory. Ramulator is capable of simulating standard <em>DDRx </em>memories (e.g., <em>DDR3-config.cfg</em>, <em>DDR4-config.cfg</em>, <em>GDDR5-config.cfg</em>), new 3D-stacked memories (e.g., <em>HBM-config.cfg</em>, <em>WideIO2-config.cfg</em>), Non-Volatile Emerging Memory Technologies (e.g., <em>PCMconfig.cfg</em>, <em>STTMRAM-config.cfg</em>), and academic proposals (e.g., <em>SALP-config.cfg</em>, <em>DSARP-config.cfg</em>). For a recent study that shows Ramulator’s capabilities in evaluating workload-DRAM configurations, please see the SIGMETRICS 2019 paper entitled <em>”Demystifying Complex Workload-DRAM Interactions: An Experimental Study” </em>by Ghose et al. [6].

In this assignment, you will simulate a DDR4-based main memory. If you open the DDR4 configuration file, you will see several parameters that describe the system to simulate. Here is a description of the most important parameters:

<ol>

 <li><sub>Memory-specific parameters: </sub>These are parameters that specify how the memory device will be configured. Important parameters are:

  <ul>

   <li><em><sub>standard</sub></em>: Specifies the DRAM standard (e.g., <em><sub>DDR4</sub></em>).</li>

   <li><em><sub>channels, ranks</sub></em>: These specify the number of DRAM channels and ranks per channel of the simulated DRAM subsystem.</li>

   <li><em><sub>speed</sub></em>: This parameter specifies the timing properties of the simulated DRAM device (e.g., <em>DDR4 </em><em><sub>2400R</sub></em>). It sets the frequency of the DRAM device and internal DRAM timings.</li>

   <li><em><sub>org</sub></em>: Specifies internal organization of the DRAM device (e.g., number of banks, rows, columns).</li>

  </ul></li>

</ol>

<table width="35">

 <tbody>

  <tr>

   <td width="14">no</td>

   <td width="21"> </td>

  </tr>

  <tr>

   <td width="14"> </td>

   <td width="21">all</td>

  </tr>

 </tbody>

</table>

<ol start="2">

 <li><sub>Core-specific parameters: </sub>These specify the configuration of the CPU cores. Ramulator implements a simple out-of-order core model, and a non-coherent cache hierarchy. The number of cores is determined by the number of trace files passed as arguments during execution (e.g., the first trace is assigned to the first core). The cache hierarchy can be configured using the cache knob with for no cache, L1L2 for private L1 and L2 caches, L3 for only L3 cache shared among the cores, and for private L1 and L2 and shared L3. In this assignment, you should not instantiate any caches since we provide you cache-filtered traces.</li>

</ol>

<table width="24">

 <tbody>

  <tr>

   <td width="24">off</td>

  </tr>

 </tbody>

</table>

<ol start="3">

 <li>Simulation-specific parameters: These parameters control the simulation. Since simulation of a program can take orders of magnitude more time to complete than executing the same program on a real system, one common practice is to limit the total number of instructions simulated (ex pected limit insts). Setting this parameter will cause the simulation to finish as soon as <em><sub>all </sub></em>cores retire at least expected limit insts instructions. Please make sure to keep early exit =as otherwise the simulation will finish when <em><sub>any </sub></em>core retires expected limit insts number of instructions.</li>

</ol>

<h2>2.5. Simulating Multi-programmed Workloads</h2>

Ramulator allows simulating multi-programmed workloads. A multi-programmed workload is composed of multiple individual applications, each of which is assigned to a CPU core. These applications do not communicate with each other. Therefore, a cache coherence mechanism is not required. However, the individual applications still interfere with each other at different levels of the memory hierarchy since they share the memory sub-system.

To create a multi-programmed simulation, you just need to specify multiple trace files in the command line, separated by space. Ramulator assigns each trace to a different core. For example, if you provide two trace files (e.g., trace1 trace2), Ramulator will automatically instantiate two CPU cores and assign each trace to a different core.

In the <em><sub>‘./traces/’ </sub></em>directory, we provide two CPU trace files. One of them represents an application that accesses main memory a lot (i.e., it has high memory intensity) and the other makes fewer requests to main memory.

<table width="585">

 <tbody>

  <tr>

   <td width="585">Exercise 5/6: Running the multi-programmed workload:The following command starts simulation with one instance of each trace file we provide in the <em><sub>‘./traces/’ </sub></em>directory.$ ./ramulator configs/DDR4-config.cfg –mode=cpu –stats  multi-programmed-simulation.stats ./traces/high-mem-intensity.trace  ./traces/low-mem-intensity.trace</td>

  </tr>

 </tbody>

</table>

Note that, in the stats file, some of the statistics are collected and displayed in the output separately for each core.

<h2>2.6. Code Organization</h2>

Ramulator abstracts basics DRAM operations to provide an easy-to-extend design. You can read Section 2 in the Ramulator paper [3] to understand how the code is organized in detail.

In this section, we give a high-level view of how different Ramulator modules communicate with each other. Figure 1 provides a simplified view of Ramulator’s functionalities. We describe CPU Trace Driven simulation in two simulation phases: processor and memory.

Processor-side simulation. The processor-side phase of the simulation consists of 1) reading the trace file; 2) issuing bubble instructions (i.e., non-memory instructions); 3) issuing memory instructions. The processor implements a simple out-of-order core model in <em><sub>‘src/Core.h’ </sub></em>and a cache subsystem in <em><sub>‘src/Cache.h’</sub></em>. It works as follows.

<ol>

 <li><em><sub>‘src/Main.cpp’ </sub></em>reads the configuration file and instantiates the processor cores and memory controllers. The number of core objects created depends on the number of trace files passed to the simulation and the number of controller objects created depends on the DRAM channels in the configuration.</li>

 <li>run cputrace() starts running, and then controlling the execution of the simulation.</li>

 <li>The processor module is ticked (tick()), which advances the simulation to the next clock cycle.</li>

</ol>

Figure 1. Sequence diagram describing how Ramulator operates.

<table width="67">

 <tbody>

  <tr>

   <td width="27">call</td>

   <td width="40">back()</td>

  </tr>

 </tbody>

</table>

<ol start="4">

 <li>The processor ticks the cores and the cache subsystem. On each clock cycle, each core reads instructions from its trace file. When a core receives a memory instruction from the simulated trace, the core calls send() to forward a memory request to the corresponding cache (if the configuration file defines caches) or memory controller. send() implements the functionality to determine which controller should service the request. Then, once a controller completes servicing a request, it informs the core that issued the request by calling, which is a function passed along with the request object as a member variable.</li>

</ol>

Memory-side simulation. The memory-side phase of the simulation has three main tasks: 1) to serve completed reads; 2) to refresh the memory device; 3) to schedule DRAM commands. The memory module implements a memory controller (one for each DRAM channel) that is responsible for performing these three tasks. The memory-side simulation works as follows.

<table width="87">

 <tbody>

  <tr>

   <td width="20">mem</td>

   <td width="67">ory.tick()</td>

  </tr>

 </tbody>

</table>

<ol>

 <li><em><sub>‘src/Main.cpp’ </sub></em>ticks the memory module (), which advances the simulation to the next clock cycle. The memory module then ticks each memory controller.</li>

 <li><sub>Serving completed reads. </sub>The controller checks if any request is completed, and if so, it informs the core that issued the requests by calling the corresponding callback(req) function.</li>

 <li>Refreshing the memory device. The controller calls the refresh module, which is responsible for issuing refresh commands at the defined refresh interval.</li>

</ol>

<table width="54">

 <tbody>

  <tr>

   <td width="14">AC</td>

   <td width="13">TI</td>

   <td width="27">VATE</td>

  </tr>

 </tbody>

</table>

<table width="114">

 <tbody>

  <tr>

   <td width="27">chan</td>

   <td width="20">nel</td>

   <td width="6">–</td>

   <td width="20">&gt;up</td>

   <td width="40">date()</td>

  </tr>

 </tbody>

</table>

<table width="67">

 <tbody>

  <tr>

   <td width="47">SpeedEn</td>

   <td width="20">try</td>

  </tr>

 </tbody>

</table>

<ol start="4">

 <li>Scheduling DRAM commands. The memory scheduler determines which request in the memory request queue should be serviced next. The selected request depends on the scheduling policy that is in use. Every cycle, the scheduler uses the scheduling policy to scan the requests in the queue to find the most appropriate request to service next, as dictated by the scheduling policy in use. If there is any such request ready to be serviced, the scheduler forwards this request to the controller, and the controller issues the appropriate DRAM command (e.g., READ,) for servicing the request. Then, the the controller callsto update the state of the DRAM device to reflect the effect of the issued DRAM command. It is important to point out that the timing of each DRAM command is defined in the standard DRAM specification (i.e., datasheets). Each memory technology file (e.g., <em><sub>‘src/DDR4.h’</sub></em>) defines timing specifications for different device types that implement the standard. For the timing parameters, you can see thestructure in <em><sub>‘src/DDR4.h’</sub></em>.</li>

</ol>

Now, let’s see how different memory scheduling policies impact performance. Open <em><sub>‘src/Scheduler.h’ </sub></em>with your favorite text editor. By default, Ramulator employs the <em><sub>FRFCFS Cap </sub></em>scheduler [7]. You can change this scheduler by modifying the line that is shown below: <strong>type </strong>= Type::FRFCFS_Cap; //Change this line to change scheduling policy

Exercise 6/6: Modifying Ramulator source code:

Change the appropriate source files to make Ramulator to use the <em><sub>FCFS</sub> </em>(First-Come First-Serve) memory scheduler. After the changes, run make again to compile Ramulator with the latest changes. Then, simulate the high memory intensity trace and compare the execution time of <em><sub>FCFS</sub> </em>to the default <em><sub>FRFCFS</sub> </em><em><sub>Cap </sub></em>policy.

<h1>3. Your Task 1/3: Implementing ATLAS</h1>

Your goal is to extend Ramulator by implementing the <em>Adaptive per-Thread Least-Attained-Service (ATLAS) </em>scheduler [1]. The key idea of ATLAS is to periodically order threads based on the service they have attained from the memory controllers, and prioritize threads that have attained the least service compared to the others in each period. This technique significantly reduces the time the CPU cores stall and, as a result, improves system throughput, as shown by Kim et al. [1].

Your task is to extend Ramulator with the ATLAS scheduling policy, as described in Sections 3-5 in the paper that proposed ATLAS [1]. Although you should stick to the exact mechanisms described in the paper, it is your task to figure out how to implement ATLAS in Ramulator. There are multiple ways to extend Ramulator with a new memory scheduling policy. For example, you can add ATLAS as a new scheduler type in <em><sub>‘Sched</sub>uler.h’ </em>(similar to other policies such as <em><sub>FCFS</sub></em>) and modify/add functions in that header file. <sub>You will not </sub>be provided with a specific software design and you are free to implement ATLAS in Ramulator as you find appropriate.

Although we do not restrict you to a specific software implementation, you should make sure your ATLAS implementation is functionally equivalent to the mechanism described in the paper. Also, please use the default configuration of the ATLAS mechanism that is provided in the paper at the end of Section 6 (i.e., <em>quantum length </em>= 10 million cycles, <em><sub>α </sub></em>= 0.875, and <em><sub>T </sub></em>= 100K cycles).

<h1>4. Your Task 2/3: Implementing BLISS</h1>

In this second task, your goal is to extend Ramulator by implementing the <em><sub>BLISS </sub></em>scheduler [2]. The key idea of BLISS is to separate applications in two groups, one containing application with high memory intensity and another that includes applications that access the memory less. The BLISS scheduler achieves its grouping by identifying applications that access a row many times in repetition and deprioritizing them for a determined amount of time. As shown by Subramanian et al. [2], BLISS reduces the interference between the two groups and improves system throughput and fairness.

Your task is to extend Ramulator with the BLISS scheduling policy, as described in Sections 4-5 in the paper that proposed BLISS [2]. Similar to Task 1, you will not be provided a specific way of implementation in Ramulator and you are free to implement BLISS in Ramulator as you find appropriate.

Although we do not restrict you to a specific software implementation, you should make sure your BLISS implementation is functionally equivalent to the mechanism described in the paper. Also, please use the default configuration of the BLISS mechanism provided in the paper at the end of Section 6.5 (i.e., <em><sub>Blacklisting </sub>Threshold </em>= 4, <em>Clearing Interval </em>= 10K cycles).

<h1>5. Your Task 3/3: Evaluating ATLAS and BLISS and Comparing Them to Conventional Memory Scheduling Policies</h1>

Your task is now to evaluate the <em>instruction throughput (IT) </em>and <em>maximum slowdown (MS) </em>that your ATLAS and BLISS implementations provide compared to three baseline scheduling policies: FCFS, FRFCFS, and FRFCFS Cap. Use the following definitions of <em>instruction throughput (IT) </em>and <em>maximum slowdown (MS)</em>:

which is basically the sum of all instructions retired in each core divided by the total number of CPU cycles the simulation took to complete.

To calculate the slowdown of a single application, simply divide the execution time of the application when running <em><sub>together </sub></em>with other applications in the same system by the execution time of the application when running the application <em><sub>alone </sub></em>on the same system. Maximum slowdown (MS) is the maximum of the single application slowdowns within a multi-programmed workload.

As a first part of the evaluation, you will have to add instruction throughput as a statistic to Ramulator such that the output <em><sub>‘.stats’ </sub></em>contains a new <em><sub>instruction throughput </sub></em>line. To do so, you will first need to use the <em>ScalarStat </em>class and create an instance of it in a way similar to other statistics that already exist in Ramulator. You must calculate instruction throughput, as defined in the equation above.

It is not possible to directly add <em><sub>MS </sub></em>as a statistic to Ramulator as it is required to run Ramulator multiple times (once with all applications together, and once the target application alone) to collect the required information to calculate MS. Thus, you will need to calculate MS manually or write a script that will read multiple Ramulator stats files and return the MS of each application. Do the following when running the simulations:

<ol>

 <li>Make sure you do not change parts of the processor and memory configuration other than those specifically mentioned that you can change in this assignment.</li>

 <li>Run simulations until every core retires 20 million instructions.</li>

 <li>For each scheduling policy, run the following multi-programmed workloads:

  <ul>

   <li>Workload 1: HLLL (four-core)</li>

   <li>Workload 2: HHLL (four-core)</li>

   <li>Workload 3: HHHH (four-core)</li>

   <li>Workload 4: HHHHHHHH (eight-core) where H stands for an instance of the trace with high memory intensity and L stands for the trace with low memory intensity. We provide both traces, as explained in Section 2.1.</li>

  </ul></li>

</ol>

Run Ramulator using the following memory schedulers, collect the instruction throughput and MS of these runs, and analyze the results.

<ul>

 <li>FCFS (First-Come First-Serve): The first memory request to be inserted into the memory request queue is serviced first.</li>

 <li>FRFCFS (First-Ready First-Come First-Serve): Similar to FCFS but requests in the request queue that target already open rows are prioritized.</li>

 <li><sub>FRFCFS Cap: </sub>Similar to FRFCFS but the number of read/write requests that can be serviced from an already open row is limited to prevent starvation of other requests that target different rows in the same bank. In other words, with this policy, once a row is activated, it could only serve a certain number of read/write requests and then it must be closed (do not change the default parameter of FRFCFS Cap). See [7] for a more detailed description and evaluation of this policy.</li>

 <li><sub>ATLAS: </sub>This is your implementation of the ATLAS scheduler described in [1].</li>

 <li><sub>BLISS: </sub>This is your implementation of the BLISS scheduler described in [2].</li>

</ul>

Evaluate each workload using each scheduling policy listed above. Collect the <em><sub>instruction throughput (IT) </sub></em>and <em><sub>maximum slowdown (MS) </sub></em>results and plot them as shown in the template in Figure 2. Note that you should show IT and MS results in separate graphs for all five scheduling policies and four multi-programmed workloads.

Based on your analysis, submit answers to the following questions with your lab report.

<ol>

 <li>Provide two graphs, one for IT and another for MS, depicting the metrics for 4 different workloads and 5 different scheduling policies.</li>

 <li>Explain the two graphs.</li>

 <li>How does each of the throughput and MS metrics change when using each scheduling policy? Explain why.</li>

 <li>Do the results match your expectations? Clearly explain what kind of difference each scheduling policy you expect to make. If the results do not match your expectations, try to reason why you may not be seeing the expected results.</li>

</ol>

Figure 2. A template showing how <em>IT </em>and <em>MS </em>results should be plotted.

<h1>6. Bonus Task: Designing Your Own Memory Scheduler</h1>

In this task, your goal is to come up with a <em><sub>new </sub></em>memory scheduling idea (or multiple ones) that hopefully performs better than the existing five scheduling policies. To generate a new idea, you may want to find and study prior work in more detail and cover the research in the area. Alternatively, you can exercise your creativity and insight. You are free to come up with any kind of memory scheduling idea as long as it is your own.

Evaluate your idea in a way similar to how you evaluated ATLAS and BLISS in Task 3. Compare your new idea against all five scheduling policies we mentioned.

Submit 1) a detailed description of your idea, 2) Ramulator implementation of the idea, and 3) the results of your new policy. You may create graphs similar to those you created for Task 3.

You can receive 1.5% of the entire course grade if you come up with a good memory scheduling idea that outperforms the five scheduling policies mentioned in this assignment. You may also receive credit for particularly creative and insightful ideas.

<h1>7. Tips</h1>

<ul>

 <li>Please do not distribute the provided program files. These are for exclusive individual use of each student of the Computer Architecture course. Distribution and sharing violates the copyright of the software provided to you.</li>

 <li>Read this handout in detail.</li>

 <li>If needed, please ask questions to the TAs using the online Q&amp;A forum in Moodle.</li>

 <li>When you encounter a technical problem, please first read the error messages. A search on the web can usually solve many debugging issues, and error messages.</li>

</ul>