# Annex B Lesson 1 – Operating Systems: Knowledge Check

**Lesson:** DS.02.01 – Operating Systems and Capabilities
**Course:** Data Systems Administrator Course (M09CVQ1)
**Source:** *Annex B Lesson 1 – Operating Systems* student outline (rev. 20230920)

> [!info] How to use this check
> Try each question before you open the answer. Click a collapsed **Answer** callout to see the answer and why it's correct. Questions follow the order of the student outline.

---

## 1. Characteristics of Operating Systems

### 1.1 What is an Operating System?

**Q1. What is an operating system?**

> [!success]- Answer
> **Software that acts as the interface between the user and the computer.** It handles memory management, file management, input/output, security, process management, job accounting, error detection, system performance control, and control of peripheral devices such as printers and disk drives.
>
> **Explanation:** The key word is *interface*. Users and applications don't talk to the hardware directly. The OS sits between them and translates requests into hardware operations.

**Q2. Why is the OS called the most significant program on a computer or server?**

> [!success]- Answer
> **Every device relies on the OS to run all of its other programs and applications.**
>
> **Explanation:** Applications are built to run *on top of* an OS. Without it, no program can take keyboard input, show output on the screen, track files, or use storage and peripherals.

**Q3. List four of the main tasks the OS performs on a computer or server.**

> [!success]- Answer
> Any four of:
> - Reading input from the keyboard
> - Sending output to the screen
> - Tracking files
> - Managing storage drives
> - Controlling peripheral devices (e.g., printers)
>
> **Explanation:** These are the everyday tasks the outline names. Each one is a way the OS stands between the user and the hardware.

### 1.2 Components of an Operating System

**Q4. Which OS component loads first and stays in main memory?**

> [!success]- Answer
> **The kernel.**
>
> **Explanation:** The kernel provides the basic level of control over all computer peripherals. It stays in RAM so it can manage memory access for programs, broker their access to hardware resources, and reset CPU operating states for the best performance.

**Q5. What is an interrupt, and what happens when the hardware receives one?**

> [!success]- Answer
> **An interrupt is a signal from a device or a program that requires the OS to stop and decide what to do next.** When one arrives, the hardware automatically pauses the running program, saves its status, and runs the program already associated with that interrupt.
>
> **Explanation:** Interrupts give the OS a reliable way to communicate with and react to its surroundings, such as a key press or a disk finishing a read, without constantly checking every device.

**Q6. Which component moves processes back and forth between disk and main memory, and decides which process gets memory and when?**

> [!success]- Answer
> **Memory management.**
>
> **Explanation:** Memory management tracks every memory location (allocated or free), decides how much memory each process may have, and updates the status when memory is freed. It is divided into three groups: **hardware**, **OS**, and **application** memory management.

**Q7. How does an OS let multiple programs appear to run at the same time on a system that can only perform one or two tasks at once?**

> [!success]- Answer
> **Time-sharing (multitasking).**
>
> **Explanation:** In multitasking, each program gets a slice of the CPU's time. Because the OS switches between programs so quickly, several independent programs appear to run at the same time.

**Q8. What does the Process Execution component do?**

> [!success]- Answer
> **It gives application programs an interface to the hardware.** Programs reach hardware by following procedures configured into the OS, and the kernel creates a process that uses memory and other resources.
>
> **Explanation:** Process execution is how a program actually gets to run. The kernel creates the process and hands it the resources it needs.

**Q9. According to the outline, what does a high-protection system offer beyond simply allowing or denying requests?**

> [!success]- Answer
> **Auditing options.**
>
> **Explanation:** The security component must tell apart requests that should be allowed from those that shouldn't. A highly protected system also *audits*, which lets administrators monitor who requested access to which resources. This ties directly to STIG and cybersecurity-policy compliance.

**Q10. What are the two types of user interface (UI)?**

> [!success]- Answer
> **GUI (Graphical User Interface)** and **CLI (Command Line Interface)**.
>
> **Explanation:** A CLI is text-based: commands are typed on a command line with a keyboard. A GUI uses graphical elements. In both cases the UI's main job is to take input from the operator and give output back.

**Q11. What does the Networking component let computers running *different* operating systems do?**

> [!success]- Answer
> **Join a common network and share resources** such as data, computing power, scanners, and printers over wired or wireless connections.
>
> **Explanation:** Network design has to account for routing, connection methods, and security. Modern operating systems support many networking techniques, so mixed-OS environments like the MCEN can interoperate.

**Q12. Per the outline's summary note, what are the main components of an OS?**

> [!success]- Answer
> **Kernel, API (Application Program Interface), user interface and file system, hardware devices, and device drivers.**
>
> **Explanation:** This summary list sits at the end of the components section and is a likely test item. Learn it alongside the eight detailed components (kernel, process execution, interrupt, memory management, multitasking, networking, security, user interface).

### 1.3 Purpose of an Operating System & Hardware

**Q13. What is the purpose of an operating system?**

> [!success]- Answer
> **It manages the computer's memory, processes, software, and hardware, and it lets people communicate with the computer without knowing the device's language.**
>
> **Explanation:** The outline puts it bluntly: *without an operating system, a computer or server is useless.*

**Q14. Which hardware component executes program instructions and performs arithmetic, logic, control, and I/O operations?**

> [!success]- Answer
> **The CPU (Central Processing Unit).**
>
> **Explanation:** The CPU, also called the processor or main processor, is the circuitry that carries out a program's instructions. It is separate from main memory, I/O circuitry, and specialized processors such as GPUs.

**Q15. What makes RAM different from storage such as hard disks, CD-RWs, or magnetic tape?**

> [!success]- Answer
> **RAM can read or write data in almost the same amount of time no matter where the data is physically located.**
>
> **Explanation:** On other direct-access media, access time depends on where the data sits, because of mechanical limits like platter rotation and arm movement. RAM holds working data and machine code.

**Q16. Is an HDD volatile or non-volatile? How does it store data?**

> [!success]- Answer
> **Non-volatile.** It stores data magnetically on rigid, rapidly spinning platters, which magnetic heads on a moving actuator arm read and write.
>
> **Explanation:** Non-volatile means the data stays when the power is off. Data is accessed randomly, so blocks can be stored and retrieved in any order.

**Q17. What is the key physical difference between an SSD and an HDD?**

> [!success]- Answer
> **An SSD has no spinning disks or moving read/write heads.** It stores data persistently on integrated circuits, usually flash memory.
>
> **Explanation:** Both serve as secondary storage, but an SSD is purely solid-state, while an HDD is electro-mechanical.

**Q18. What is RAID, and what two purposes does it serve?**

> [!success]- Answer
> **Redundant Array of Independent Disks.** It is a storage virtualization technology that combines multiple physical drives into one or more logical units for **data redundancy**, **performance improvement**, or both.
>
> **Explanation:** RAID replaced the older idea of one highly reliable, expensive mainframe disk, called **SLED (Single Large Expensive Disk)**.

---

## 2. Types of Operating Systems

**Q19. What are the five main types of operating systems covered in this lesson?**

> [!success]- Answer
> 1. Microsoft Windows
> 2. Apple macOS
> 3. Google's Android OS
> 4. Apple iOS
> 5. Linux
>
> **Explanation:** These five run most of what a data center controls: servers, phones, computers, and tablets. Knowing them helps administrators maintain security and user access and perform routine operations.

**Q20. What is the difference between a *workstation* and a *server*?**

> [!success]- Answer
> **A workstation is a high-end personal computer** for CPU- and RAM-heavy work (graphic design, video editing, CAD, 3-D design), usually with a fast processor, multiple drives, lots of RAM, and sometimes special cards. **A server is an application or device that provides services to connected clients** in a client-server architecture.
>
> **Explanation:** Manufacturers market workstations to professional users. A server is more of a *utility* device: it shares files and services and gives administrators control of networks, data storage, and applications.

**Q21. What does it mean that Windows and macOS are *proprietary* operating systems?**

> [!success]- Answer
> **The company conceptualized, designed, developed, and sells its own OS, and users aren't meant to tamper with or modify it.**
>
> **Explanation:** Compare this with Linux, which is open source.

**Q22. Is Apple macOS currently used on USMC networks?**

> [!success]- Answer
> **No.** The outline notes that macOS has not made its way onto USMC networks yet, though it could in the future.

**Q23. Android OS is based on what, and where is it used within the USMC?**

> [!success]- Answer
> **It is based on a Linux distribution and other open source software.** It is used mainly within the **USMC Mobility Infrastructure**.
>
> **Explanation:** This matters because many junior enlisted Marines sent to Base G-6s must support end users' mobile devices.

**Q24. Which Apple mobile OS operates on the Marine Corps Enterprise Network, and under what infrastructure does it fall?**

> [!success]- Answer
> **Apple iOS (iPhones), under the Mobility Infrastructure.**
>
> **Explanation:** Like Android, iOS devices now operate on the MCEN. iOS runs only on iPhones and is known for touch gestures and easy connections to other Apple devices.

**Q25. What makes Linux different from Windows and macOS? Name the Linux-based systems found on Marine Corps networks.**

> [!success]- Answer
> **Linux is not proprietary. It is a family of free, open source systems that anyone can modify and distribute.** Linux-based systems on Marine Corps networks include **Cisco Call Manager**, **ACAS**, and some **remote administration servers**.
>
> **Explanation:** Linux is popular because it is easy to customize and offers many options to people who know how to use it.

---

## 3. Types of Operating System Applications

**Q26. What is the main difference between an operating system and application software?**

> [!success]- Answer
> **An OS is *system software* that works as the interface between the user and the hardware. Application software is a program that performs a *specific task* for the end user.**
>
> **Explanation:** The OS runs the machine, and applications run on top of the OS to get the user's work done.

**Q27. Give two examples of application software.**

> [!success]- Answer
> Any two of: **Microsoft Word**, **Microsoft Excel**, **Mozilla Firefox**, **Google Chrome**.
>
> **Explanation:** Each one does a specific job for the end user (word processing, spreadsheets, web browsing) and doesn't manage the hardware.

**Q28. When can the operating system itself be considered application software?**

> [!success]- Answer
> **When it performs simple calculating, measuring, rendering, or word processing tasks that aren't used to control hardware through a CLI or GUI.**
>
> **Explanation:** The line is drawn by function. Controlling hardware is OS work, and task-specific work for the user is application work.

---

## 4. Servers

### 4.1 Definition of a Server

**Q29. Define a server.**

> [!success]- Answer
> **A computer program or device that provides a service to another computer program and its user, known as the client.**
>
> **Explanation:** In a data center, the physical computer the server program runs on (such as an **ESXi host**) is also often called a server. This is done through server virtualization.

**Q30. In the client/server model, who can be a "client"?**

> [!success]- Answer
> **Other users making requests, or other servers on the network** that are trying to process information from one another.
>
> **Explanation:** The server program waits for and fulfills requests from clients on the network. This is the typical model for USMC architecture.

**Q31. Give an example of a single-task server and a multi-task server.**

> [!success]- Answer
> - **Single-task:** a **mail server**, which accepts and stores email and provides it to a requesting client.
> - **Multi-task:** a **file and print server**, which stores files *and* accepts print jobs and sends them to a network-attached printer.

### 4.2 How a Server Works

**Q32. What must a device be configured to do in order to function as a server?**

> [!success]- Answer
> **Listen for requests from clients on a network connection.**
>
> **Explanation:** This capability can come from the OS, from an installed application or role, or from both. For example, Windows Server can listen and respond out of the box, and installing roles or services expands the kinds of requests it can answer.

**Q33. What is the request-and-response model also known as?**

> [!success]- Answer
> **The call-and-response model.**
>
> **Explanation:** A client sends a request over the network, and the server receives it and responds with the appropriate information.

**Q34. Name three additional tasks a server often performs during a single request and response.**

> [!success]- Answer
> 1. **Verifying the requestor's identity** (authentication)
> 2. **Making sure the client has permission** to access the requested data or resources (authorization)
> 3. **Formatting the response properly** and returning it in the expected way

---

## 5. Types of Servers

**Q35. Which server type provides an environment to develop, process, and run web-based applications? Give examples.**

> [!success]- Answer
> **Application server.** Examples: **PHP, Java, .NET Framework**.
>
> **Explanation:** Application servers can run many kinds of applications no matter what those applications do. Their advantages are processing high-end jobs and storing data centrally.

**Q36. What characterizes a cloud server, and what is cloud computing?**

> [!success]- Answer
> **Cloud servers are highly distributed, with massive storage and high-end processing power, and serve many requests at once.** **Cloud computing** is an environment reached through an application (e.g., Google Drive) or a web browser on computers, smartphones, or tablets.
>
> **Explanation:** Examples: Amazon Web Services, Google Docs, Windows Azure. **Microsoft Teams is cloud-based.** The USMC is moving to the cloud, so know the difference between **on-premise** and **off-premise** services.

**Q37. In a database server, what is the difference between the "frontend" and the "backend"?**

> [!success]- Answer
> **The frontend runs locally on the user's machine. The backend runs on the database itself** and can be reached by any remote device or shell.
>
> **Explanation:** Users get data out of a database by running a query in a query language. **SCCM and SQL servers** hold large databases that 0671s administer.

**Q38. What everyday analogy does the outline use for a mail server? Which mail platforms do USMC networks use?**

> [!success]- Answer
> **A post office**, where mail is stored and sorted before it goes to its final destination. USMC networks currently use **Microsoft Exchange 2016, 2019, and Exchange O365**.
>
> **Explanation:** Related protocols listed in the outline are **SMTP** (Simple Mail Transfer Protocol) and **POP3** (Post Office Protocol 3).

**Q39. What does a DNS server do, and which port does it use?**

> [!success]- Answer
> **It receives a request containing a domain name or hostname and responds with the matching IP address. DNS uses port 53.**
>
> **Explanation:** DNS (Domain Name System) translates human-friendly names into the IP addresses that computers use to communicate. Port 53 is a commonly tested fact.

**Q40. What two benefits does a proxy server's caching provide?**

> [!success]- Answer
> 1. **Pages load faster** for users who visit a page that was already cached.
> 2. **Less network bandwidth is used.**
>
> **Explanation:** A proxy server is part of the gateway, separating the local network from worldwide networks. It caches pages accessed through the network so it can serve them again without fetching them from the internet.

---

## 6. Operating System Installation Types

**Q41. What are the three most common Windows installation methods?**

> [!success]- Answer
> 1. **DVD Boot / .ISO installation**
> 2. **Distribution share installation**
> 3. **Image-based installation**

**Q42. What is an ISO file, and where does the name come from?**

> [!success]- Answer
> **An exact virtual copy (an "image") of a CD or DVD, with the same file structure and data.** The name comes from the **International Organization for Standardization**.
>
> **Explanation:** To install from an ISO, the administrator copies it to a USB drive or DVD, mounts it, and runs **Setup.exe** from the root folder. Changing the BIOS boot order lets the installation run directly from the USB drive or DVD at power-on, without an existing OS.

**Q43. What is a distribution share, and which file tells Windows Setup what to copy from it?**

> [!success]- Answer
> **An optional set of folders containing custom scripts, images, branding, applications, drivers, and other files.** The **answer file (Unattend.xml)** specifies what gets copied.
>
> **Explanation:** During installation, Windows connects to the server share using the credentials in the answer file. Only the files named in the answer file are copied.

**Q44. If there is no network or server share, how can you still use distribution share files?**

> [!success]- Answer
> **Copy the needed files into a configuration set** (a subset of the distribution share) on external storage such as a USB flash drive or external hard disk.
>
> **Explanation:** On USMC networks this method is the *least likely* to be used, but it is still a dependable fallback.

**Q45. Describe image-based installation and name the Microsoft tool given as an example.**

> [!success]- Answer
> **Install and configure an OS and applications on a reference system, then use an imaging tool to capture a sector-by-sector image of it for deployment.** Example: **Microsoft SCCM (System Center Configuration Manager)**.

**Q46. What is SCCM, and which Microsoft patching system does it use?**

> [!success]- Answer
> **SCCM is a paid lifecycle management solution that tracks network inventory, helps install applications, and deploys updates and security patches across a network. It uses WSUS** (Windows Server Update Services) to check for and install updates.
>
> **Explanation:** SCCM adds more control over *when* and *how* patches are applied, which makes it attractive for large enterprises. It can be challenging, though, when one tool has to patch every device, OS, and third-party application.

**Q47. What is the difference between a guided install and a manual install?**

> [!success]- Answer
> - **Guided install:** installs the OS and device drivers in **unattended** mode.
> - **Manual install:** the administrator installs the OS and device drivers **by hand**.

**Q48. Can Windows Setup perform computer-to-computer migrations? If not, what should you use?**

> [!success]- Answer
> **No.** Use **Windows Easy Transfer**, the **User State Migration Tool (USMT)**, or another migration tool to move data from the previous installation to the new OS.
>
> **Explanation:** Windows Setup handles only **clean (custom)** and **upgrade** installations on the same machine.

**Q49. What is the difference between a custom (clean) installation and an upgrade installation?**

> [!success]- Answer
> - **Custom (clean) installation:** saves the previous Windows installation but **does not migrate settings**, and the previous installation **will not boot** afterward.
> - **Upgrade installation:** **keeps your settings and preferences** while upgrading the OS.

---

## 7. Server Operating System Edition Types

**Q50. What two editions does Windows Server 2019 come in?**

> [!success]- Answer
> **Standard** and **Datacenter**.
>
> **Explanation:** Choosing the right edition matters. The wrong one either can't meet the organization's needs or adds cost for features nobody uses.

**Q51. Which earlier OS is Windows Server 2019 based on, and what new capabilities did it add?**

> [!success]- Answer
> **It is based on Windows Server 2016.** It added **hyper-converged infrastructure (HCI)** and **hybrid capabilities with Azure** (cloud services such as virtual computing, analytics, storage, and networking).
>
> **Explanation:** Server 2019 also aims to reduce business risk and improve security through multiple layers of security built into the OS.

**Q52. How many VMs does one Windows Server 2019 Standard license allow, and what happens if you need more?**

> [!success]- Answer
> **Up to 2 VMs with 1 Hyper-V host per license.** To add 2 more VMs, **you must license all the cores in the server again.**
>
> **Explanation:** That limit is why Standard suits **small and medium organizations**.

**Q53. What are the Storage Replica limits in Standard edition compared with Datacenter edition?**

> [!success]- Answer
> - **Standard:** limited to **1 partnership and 1 resource group**, and a **single volume up to 2 TB**.
> - **Datacenter:** **no limits** on the amount of data or number of volumes.
>
> **Explanation:** With unlimited replication in Datacenter, administrators can replicate or back up all their data for disaster recovery.

**Q54. Name three features Datacenter has that Standard lacks.**

> [!success]- Answer
> Any three of:
> - **Host Guardian Hyper-V Support** (makes sure the enterprise or hosting company knows about the Hyper-V hosts, and manages the keys needed to start **shielded VMs**)
> - **Software-Defined Networking (SDN)** (program, monitor, and improve the network from one central point)
> - **Storage Spaces Direct** (highly available software-defined storage on industry-standard servers with local drives, cheaper than traditional NAS or SAN arrays)
> - **Unlimited Storage Replica**
> - **Unlimited VMs / Windows Server instances**

**Q55. Standard edition can be activated as a guest only under what condition? How can Datacenter be activated?**

> [!success]- Answer
> **Standard** can be activated as a guest **only if the VMs are hosted on the Datacenter edition**. **Datacenter** can be activated **as either a guest or a host**.

**Q56. A command needs heavy virtualization (many VMs) and advanced storage. Which edition should it choose, and why?**

> [!success]- Answer
> **Windows Server 2019 Datacenter.**
>
> **Explanation:** Datacenter allows an **unlimited number of VMs** (1 Hyper-V host per license), unlimited Windows Server instances on the physical machine, unlimited Storage Replica, SDN, and Storage Spaces Direct. Standard's 2-VM limit and missing features would force repeated core relicensing and still wouldn't meet the requirement.

---

## Quick Reference: Commonly Tested Facts

| Topic | Fact |
|---|---|
| First component to load | Kernel |
| UI types | GUI and CLI |
| Older single-disk concept RAID replaced | SLED (Single Large Expensive Disk) |
| HDD vs. SSD | Both non-volatile; HDD has moving parts, SSD uses flash memory |
| Five OS types | Windows, macOS, Android, iOS, Linux |
| OS not yet on USMC networks | macOS |
| Linux on USMC networks | Cisco Call Manager, ACAS, remote admin servers |
| Mobile OS infrastructure | Mobility Infrastructure (Android & iOS) |
| Request/response a.k.a. | Call-and-response model |
| DNS port | 53 |
| USMC mail platforms | Exchange 2016, 2019, O365 |
| Cloud-based USMC app named | Microsoft Teams |
| 3 common install methods | DVD/ISO, Distribution share, Image-based |
| Answer file | Unattend.xml |
| SCCM patches via | WSUS |
| Migration tools | Windows Easy Transfer, USMT |
| Server 2019 editions | Standard (2 VMs/license), Datacenter (unlimited VMs) |
| Standard Storage Replica limit | 1 partnership, 1 resource group, single volume up to 2 TB |
