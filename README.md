# Benchmarks
A playground for experimenting with different server models.

# Environment
We're using the following physical machines to perform these tests:

| Name | OS | Role | CPU | RAM | NIC | Notes |
| ---- | --- | ---- | --- | --- | --- | ----- |
| perfsvr | Windows Server 2012 R2 | Web Server | [Xeon E5-1650](http://ark.intel.com/products/64601/Intel-Xeon-Processor-E5-1650-12M-Cache-3_20-GHz-0_0-GTs-Intel-QPI) | 32 GB | [Intel® Ethernet Converged Network Adapter X540-T1 10GbE](http://ark.intel.com/products/58953/Intel-Ethernet-Converged-Network-Adapter-X540-T1) |  |
| perfsvr2 | Ubuntu 14.04 LTS | Web Server & Load Generator | [Xeon E5-1620](http://ark.intel.com/products/64621/Intel-Xeon-Processor-E5-1620-10M-Cache-3_60-GHz-0_0-GTs-Intel-QPI) | 32 GB | [Intel® Ethernet Converged Network Adapter X540-T1 10GbE](http://ark.intel.com/products/58953/Intel-Ethernet-Converged-Network-Adapter-X540-T1) |  |
| perf02 | Windows Server 2012 R2 | Load Generator | [Xeon W3550](http://ark.intel.com/products/39720/Intel-Xeon-Processor-W3550-8M-Cache-3_06-GHz-4_80-GTs-Intel-QPI) | 24 GB | Broadcom NetXtreme Gigabit Ethernet (BCM5764) |  |
| perf03 | Ubuntu 14.04 LTS | Load Generator | [Xeon W3550](http://ark.intel.com/products/39720/Intel-Xeon-Processor-W3550-8M-Cache-3_06-GHz-4_80-GTs-Intel-QPI) | 12 GB | [Intel® Ethernet Converged Network Adapter X540-T1 10GbE](http://ark.intel.com/products/58953/Intel-Ethernet-Converged-Network-Adapter-X540-T1) |  |

The machines are connected to an 8-port [Netgear XS708E](http://www.netgear.com/business/products/switches/unmanaged-plus/10g-plus-switch.aspx) 10-Gigabit switch.

# Load Generation
We're using [wrk](https://github.com/wg/wrk) to generate load from one of our Linux boxes (usually perfsvr2). We also have [WCAT](http://www.iis.net/downloads/community/2007/05/wcat-63-(x64)) set up on perf02 but it as it doesn't support HTTP pipelining we've stopped using it for now.

# Results
For each stack, variations of the load parameters and multiple runs are tested and the highest result is recorded.

## Plain Text

Similar to the plain text benchmark in the TechEmpower tests. Intended to highlight the HTTP efficiency of the server & stack. Implementations are free to cache aggressively and remove/disable components that aren't required in order to maximize performance.

| Stack | Server |  Req/sec | Load Params | Impl | Observations |
| ----- | ------ | -------- | ----------- | ---- | ------------ |
| ASP.NET 4.6 | perfsvr | 65,383 | 8 threads, 512 connections | Generic reusable handler, unused IIS modules removed | CPU is 100%, almost exclusively in user mode |
| IIS Static File (kernel cached) | perfsvr | 276,727 | 8 threads, 512 connections | hello.html containing "HelloWorld" | CPU is 36%, almost exclusively in kernel mode |
| IIS Static File (non-kernel cached) | perfsvr |231,609 | 8 threads, 512 connections | hello.html containing "HelloWorld" | CPU is 100%, almost exclusively in user mode |
| ASP.NET 5 on WebListener (kernel cached) | perfsvr |264,117 | 8 threads, 512 connections | Just app.Run() | CPU is 36%, almost exclusively in kernel mode |
| ASP.NET 5 on WebListener (non-kernel cached) | perfsvr | 107,315 | 8 threads, 512 connections | Just app.Run() | CPU is 100%, mostly in user mode |
| ASP.NET 5 on IIS (Helios) (non-kernel cached) | perfsvr | 109,560 | 8 threads, 512 connections | Just app.Run() | CPU is 100%, mostly in user mode |
| NodeJS | perfsvr | 96,558 | 8 threads, 1024 connections | The actual TechEmpower NodeJS app | CPU is 100%, almost exclusively in user mode |
| Scala | perfsvr | 204,009 | 8 threads, 1024 connections |  The actual TechEmpower Scala plain text app | CPU is 68%, mostly in kernel mode |
| libuv C# | perfsvr | 300,507 | 12 threads, 1024 connections | Simple TCP server, not real HTTP yet, load spread across 12 ports (port/thread/CPU) | CPU is 54%, mostly in kernel mode | mode, NIC saturated |

## Plain Text with HTTP Pipelining

Like the Plain Text scenario above but with HTTP pipelining enabled. Only stacks/servers that support pipelining are included.

| Stack | Server |  Req/sec | Load Params | Impl | Observations |
| ----- | ------ | -------- | ----------- | ---- | ------------ |
| NodeJS | perfsvr | 148,934 | 8 threads, 1024 connections | The actual TechEmpower NodeJS app | CPU is 100%, almost exclusively in user mode |
| Scala | perfsvr | 764,775 | 8 threads, 1024 connections, pipelining 15 deep |  The actual TechEmpower Scala plain text app | CPU is 46%, mostly in kernel mode, 1Gbps NIC saturated (need to re-run on 10GbE) |
| libuv C# | perfsvr | 2,379,267 | 36 threads, 288 connections, pipelining 15 deep | Simple TCP server, not real HTTP yet, load spread across 12 ports (port/thread/CPU) | CPU is 100%, mostly in user mode, gets slower as test runs longer, issue somewhere |

## Plain Text with HTTP 2

Coming soon...

## JSON
Coming soon...

-----------------

This project is part of ASP.NET 5. You can find samples, documentation and getting started instructions for ASP.NET 5 at the [Home](https://github.com/aspnet/home) repo.


