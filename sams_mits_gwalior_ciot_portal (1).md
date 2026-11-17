<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>SAMS | MITS Gwalior – Centre for Internet of Things (CIOT)</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Google Fonts: Inter & JetBrains Mono -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
  <!-- Font Awesome 6 -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>

  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'sans-serif'],
            mono: ['JetBrains Mono', 'monospace'],
          },
          colors: {
            mits: {
              navy: '#0B1D3A',
              royal: '#132E5B',
              indigo: '#1E3A8A',
              gold: '#D97706',
              amber: '#F59E0B',
              lightgold: '#FEF3C7',
              slate: '#F8FAFC'
            }
          }
        }
      }
    }
  </script>

  <style>
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #f1f5f9; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 9999px; }
    ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }

    @keyframes ticker {
      0% { transform: translateX(0); }
      100% { transform: translateX(-50%); }
    }
    .animate-ticker {
      display: inline-flex;
      white-space: nowrap;
      animation: ticker 42s linear infinite;
    }
    .animate-ticker:hover {
      animation-play-state: paused;
    }
    @media print {
      .no-print { display: none !important; }
    }
  </style>
</head>
<body class="bg-slate-50 text-slate-800 font-sans antialiased selection:bg-amber-200 selection:text-slate-900">

  <!-- GLOBAL TOAST / MESSAGE NOTIFICATION CONTAINER -->
  <div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 max-w-sm w-full pointer-events-none px-4 sm:px-0"></div>

  <!-- GLOBAL MODAL BACKDROP -->
  <div id="modal-backdrop" class="fixed inset-0 bg-slate-900/60 backdrop-blur-xs z-40 hidden items-center justify-center p-4 overflow-y-auto">
    <div id="modal-content" class="bg-white rounded-2xl shadow-2xl border border-slate-200 max-w-2xl w-full overflow-hidden transform transition-all my-auto">
      <!-- Dynamic Modal Inner Content -->
    </div>
  </div>

  <!-- MAIN APPLICATION ROOT -->
  <div id="app" class="min-h-screen flex flex-col"></div>

  <script>
    /*************************************************************************
     * SAMS (Student Academic Management System) - MITS Gwalior (CIOT)
     * Official Session: July – Dec 2026 (w.e.f. 24.08.2026)
     * Reactive Cross-Portal State + localStorage Persistence
     *************************************************************************/

    const STORAGE_KEY = "MITS_SAMS_CIOT_2026_V5";
    const CURRENT_DATE_STR = "Tuesday, November 17, 2026";

    const DEFAULT_STATE = {
      currentPortal: 'landing', // 'landing' | 'student' | 'faculty' | 'club'
      studentTab: 'dashboard',
      facultyTab: 'attendance',
      clubTab: 'create-event',
      sidebarOpen: false,

      selectedDay: 'Tuesday',
      selectedBatch: 'Batch A', // 'Batch A' | 'Batch B' (Only affects Lab sessions)
      selectedCourseCode: '33261101',
      clubCategoryFilter: 'All',
      servicesSubTab: 'fees', // 'fees' | 'scholarships' | 'bonafide' | 'admit-help'
      feeSubTab: 'sem1',
      facultyCohortMode: 'all80', // 'all80' | 'batchA' | 'batchB'
      facultyMarksTier: 'classQuiz', // 'classQuiz' | 'centralQuiz' | 'minor' | 'major'
      activeQuizIndex: 0,

      announcements: [
        "📢 Live In-Class Quiz active for 33261101 Introduction to Electrical and Computer Engineering — Attempt in Live Quizzes tab!",
        "🏆 Micro-Project-I (33261109) Expo scheduled for Nov 25–27, 2026 at CIOT Atrium & Seminar Hall.",
        "📝 Theory Minor Exams (Nov 2–6, 2026) evaluation in progress — Result Pending.",
        "🎓 Semester Proficiency (33261108) Presentations scheduled Nov 27–29, 2026 under Dr. Namita Arya.",
        "💰 MP Post-Matric & Mukhyamantri Medhavi Chhatra Yojana (MMVY) scholarship verification open in Fees & Scholarships Hub!"
      ],

      studentProfile: {
        name: "Ashi Golhani",
        enrollment: "BTEC26O1022",
        email: "btec26o1022@mitsgwl.ac.in",
        fatherName: "Advocate Anil Golhani",
        branch: "Electrical and Computer Engineering (EACE)",
        semester: "1st Sem (July – Dec 2026)",
        department: "Centre for Internet of Things (CIOT)",
        batch: "Batch A",
        hostel: "Girls' Hostel No. 6",
        dean: "Dr. Praveen Bansal",
        coordinators: ["Dr. Abhishek Sharma", "Dr. Nand Kishore Joshi"],
        projectSupervisor: "Dr. Abhishek Sharma",
        phone: "+91 98264-71920"
      },

      facultyProfile: {
        name: "Dr. Abhishek Sharma (AS)",
        designation: "Associate Professor & Branch Coordinator (EACE)",
        department: "Centre for Internet of Things (CIOT)",
        email: "abhishek.sharma@mitsgwl.ac.in",
        selectedCourse: "33261102"
      },

      clubProfile: {
        clubName: "MITS ACM Student Chapter",
        role: "President",
        bearerName: "Ananya Deshmukh (3rd Yr, CIOT)",
        email: "president.acm@mitsgwl.ac.in",
        facultySupervisor: "Dr. Abhishek Sharma"
      },

      courses: [
        {
          code: "33261101",
          name: "Introduction to Electrical and Computer Engineering",
          shortName: "Intro to ECE",
          type: "Theory",
          credits: 4,
          faculty: "Dr. Saurabh Kumar Rajput (SKR)",
          attended: 35,
          total: 40,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 18,
          classQuizMax: 20,
          centralQuiz: 19,
          minorMarks: null,
          minorStatus: "Result Pending",
          majorMarks: null,
          majorStatus: "Starts Dec 10, 2026",
          units: [
            { unit: 1, title: "Overview of Electrical & Computer Systems", topics: "Evolution of ECE, smart grid systems, IoT sensors and embedded computing foundations." },
            { unit: 2, title: "DC & AC Circuit Fundamentals", topics: "Ohm's law, Kirchhoff's laws, phasor analysis, single-phase and three-phase circuits." },
            { unit: 3, title: "Semiconductor Devices & Diodes", topics: "PN junction diodes, rectifiers, Zener regulators, BJT and MOSFET basics." },
            { unit: 4, title: "Digital Systems & Microcontroller Interfacing", topics: "Logic gates, Boolean algebra, GPIO ports, ADC and PWM fundamentals." },
            { unit: 5, title: "Modern IoT & Edge Computing Paradigms", topics: "Sensor nodes, MQTT messaging, cloud telemetry and smart campus use-cases." }
          ],
          materials: [
            { unit: 1, title: "Unit 1 ECE Foundations & CIOT Roadmap", type: "PDF", size: "2.8 MB", date: "Aug 26, 2026", by: "Dr. Saurabh Kumar Rajput (SKR)" },
            { unit: 2, title: "Circuit Analysis Handout & Solved Problems", type: "Notes", size: "2.1 MB", date: "Sep 14, 2026", by: "Dr. Saurabh Kumar Rajput (SKR)" },
            { unit: 3, title: "Semiconductor Diodes & Transistors Question Bank", type: "Question Bank", size: "1.5 MB", date: "Oct 06, 2026", by: "Dr. Saurabh Kumar Rajput (SKR)" },
            { unit: 4, title: "Microcontroller Interfacing Slides", type: "PDF", size: "3.2 MB", date: "Oct 22, 2026", by: "Dr. Saurabh Kumar Rajput (SKR)" },
            { unit: 5, title: "IoT Edge Architecture Summary", type: "Notes", size: "1.8 MB", date: "Nov 10, 2026", by: "Dr. Saurabh Kumar Rajput (SKR)" }
          ]
        },
        {
          code: "33261102",
          name: "Computer Programming",
          shortName: "Computer Programming",
          type: "Theory",
          credits: 4,
          faculty: "Dr. Abhishek Sharma (AS)",
          attended: 32,
          total: 38,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 19,
          classQuizMax: 20,
          centralQuiz: 18,
          minorMarks: null,
          minorStatus: "Result Pending",
          majorMarks: null,
          majorStatus: "Starts Dec 12, 2026",
          units: [
            { unit: 1, title: "C Language Fundamentals & I/O", topics: "Data types, format specifiers, operators, arithmetic expressions, basic input/output." },
            { unit: 2, title: "Control Flow & Modular Functions", topics: "Branching statements (if-else, switch), loops (while, for, do-while), user-defined functions, recursion." },
            { unit: 3, title: "Arrays & Strings Manipulation", topics: "1D and 2D arrays, matrix arithmetic, string handling functions, searching and sorting algorithms." },
            { unit: 4, title: "Pointers & Dynamic Memory Allocation", topics: "Pointer arithmetic, pointers to arrays, malloc, calloc, realloc, free, file handling in C." },
            { unit: 5, title: "Structures, Unions & Preprocessor Directives", topics: "User-defined data types, arrays of structures, nested structures, macros and conditional compilation." }
          ],
          materials: [
            { unit: 1, title: "C Syntax & Memory Layout Guide", type: "PDF", size: "1.9 MB", date: "Aug 27, 2026", by: "Dr. Abhishek Sharma (AS)" },
            { unit: 2, title: "Recursive Functions & Stack Traces", type: "Notes", size: "1.4 MB", date: "Sep 18, 2026", by: "Dr. Abhishek Sharma (AS)" },
            { unit: 3, title: "Sorting & Searching Algorithms Question Bank", type: "Question Bank", size: "1.6 MB", date: "Oct 10, 2026", by: "Dr. Abhishek Sharma (AS)" },
            { unit: 4, title: "Pointers & Dynamic Memory Masterclass Notes", type: "PDF", size: "2.5 MB", date: "Oct 28, 2026", by: "Dr. Abhishek Sharma (AS)" }
          ]
        },
        {
          code: "33261103",
          name: "Basics of Embedded Programming",
          shortName: "Embedded Programming",
          type: "Theory",
          credits: 3,
          faculty: "Dr. Rupam Shrivastava (RUS)",
          attended: 30,
          total: 36,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 17,
          classQuizMax: 20,
          centralQuiz: 17,
          minorMarks: null,
          minorStatus: "Result Pending",
          majorMarks: null,
          majorStatus: "Starts Dec 14, 2026",
          units: [
            { unit: 1, title: "Introduction to Embedded Systems", topics: "Embedded vs General-purpose computing, microcontrollers, real-time constraints." },
            { unit: 2, title: "GPIO Programming & Register Configuration", topics: "Input/output ports, pull-up/pull-down resistors, button debouncing, LED matrix driving." },
            { unit: 3, title: "Interrupts & Timers", topics: "Interrupt service routines (ISRs), external interrupts, hardware timers and PWM generation." },
            { unit: 4, title: "Communication Protocols (I2C, SPI, UART)", topics: "Serial communication, master-slave architecture, baud rates, sensor polling." },
            { unit: 5, title: "Low-Power Embedded Design & RTOS Basics", topics: "Sleep modes, power saving techniques, task scheduling and multitasking fundamentals." }
          ],
          materials: [
            { unit: 1, title: "Embedded Systems Architecture Handbook", type: "PDF", size: "3.4 MB", date: "Aug 29, 2026", by: "Dr. Rupam Shrivastava (RUS)" },
            { unit: 2, title: "GPIO & Register Level Programming Notes", type: "Notes", size: "2.2 MB", date: "Sep 22, 2026", by: "Dr. Rupam Shrivastava (RUS)" },
            { unit: 3, title: "UART, SPI & I2C Protocol Reference Sheet", type: "PDF", size: "2.0 MB", date: "Nov 05, 2026", by: "Dr. Rupam Shrivastava (RUS)" }
          ]
        },
        {
          code: "33261104",
          name: "Digital Logic Design",
          shortName: "Digital Logic Design",
          type: "Theory",
          credits: 3,
          faculty: "Dr. Aftab Ahmed Ansari (AAA)",
          attended: 28,
          total: 38,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 16,
          classQuizMax: 20,
          centralQuiz: 16,
          minorMarks: null,
          minorStatus: "Result Pending",
          majorMarks: null,
          majorStatus: "Starts Dec 16, 2026",
          units: [
            { unit: 1, title: "Number Systems & Boolean Algebra", topics: "Binary, octal, hexadecimal arithmetic, logic gates (AND, OR, NOT, NAND, NOR), De Morgan's laws." },
            { unit: 2, title: "Combinational Logic Design", topics: "Karnaugh Maps (K-maps) simplification, multiplexers, demultiplexers, encoders, decoders, adders." },
            { unit: 3, title: "Sequential Logic Circuits", topics: "Latches, Flip-Flops (SR, D, JK, T), master-slave configuration, excitation tables." },
            { unit: 4, title: "Registers & Counters", topics: "Shift registers, ripple counters, synchronous counters, design of state machines." },
            { unit: 5, title: "Memory Units & Programmable Logic", topics: "RAM, ROM, PLA, PAL, and FPGA introduction." }
          ],
          materials: [
            { unit: 1, title: "Boolean Algebra & K-Map Minimization", type: "PDF", size: "2.6 MB", date: "Sep 01, 2026", by: "Dr. Aftab Ahmed Ansari (AAA)" },
            { unit: 2, title: "Sequential Circuits & Flip-Flop Notes", type: "Notes", size: "1.9 MB", date: "Sep 25, 2026", by: "Dr. Aftab Ahmed Ansari (AAA)" },
            { unit: 3, title: "DLD Practice Question Bank & Karnaugh Maps", type: "Question Bank", size: "2.1 MB", date: "Oct 15, 2026", by: "Dr. Aftab Ahmed Ansari (AAA)" }
          ]
        },
        {
          code: "33261105",
          name: "Linear Algebra and Calculus",
          shortName: "Linear Algebra & Calculus",
          type: "Theory",
          credits: 3,
          faculty: "Dr. Nidhi Humnekar (NH)",
          attended: 26,
          total: 36,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 18,
          classQuizMax: 20,
          centralQuiz: 17,
          minorMarks: null,
          minorStatus: "Result Pending",
          majorMarks: null,
          majorStatus: "Starts Dec 18, 2026",
          units: [
            { unit: 1, title: "Matrices & Systems of Linear Equations", topics: "Rank of matrix, Gauss elimination, consistency, Eigenvalues, Eigenvectors, Cayley-Hamilton theorem." },
            { unit: 2, title: "Vector Spaces & Linear Transformations", topics: "Vector spaces, subspaces, linear independence, basis, dimension, matrix representation." },
            { unit: 3, title: "Differential Calculus of Several Variables", topics: "Partial differentiation, Euler's theorem, total derivative, Jacobians, maxima and minima." },
            { unit: 4, title: "Integral Calculus & Multiple Integrals", topics: "Double and triple integrals, change of order of integration, Beta and Gamma functions." },
            { unit: 5, title: "Vector Calculus", topics: "Gradient, divergence, curl, line, surface and volume integrals, Green's, Gauss and Stokes' theorems." }
          ],
          materials: [
            { unit: 1, title: "Eigenvalues & Cayley-Hamilton Theorem Notes", type: "PDF", size: "2.4 MB", date: "Sep 03, 2026", by: "Dr. Nidhi Humnekar (NH)" },
            { unit: 2, title: "Multivariable Calculus & Jacobians Handout", type: "Notes", size: "1.7 MB", date: "Oct 02, 2026", by: "Dr. Nidhi Humnekar (NH)" }
          ]
        },
        {
          code: "33261106(A)",
          name: "Computer Programming Lab",
          shortName: "Computer Programming Lab",
          type: "Lab",
          credits: 1.5,
          faculty: "Dr. Abhishek Sharma (AS) / Mr. Vinay Gupta (VG)",
          attended: 11,
          total: 12,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 9,
          classQuizMax: 10,
          centralQuiz: null,
          minorMarks: 28,
          minorStatus: "Evaluated (28/30)",
          majorMarks: null,
          majorStatus: "Starts Dec 04, 2026",
          units: [
            { unit: 1, title: "Basic C Programs & Arithmetic Operators", topics: "GCC compiler, format specifiers, conditional branching." },
            { unit: 2, title: "Loops, Patterns & Functions", topics: "Factorial, Fibonacci series, prime checking, user-defined functions." },
            { unit: 3, title: "1D/2D Arrays & Matrix Multiplication", topics: "Array traversal, transpose, multiplication, string operations." },
            { unit: 4, title: "Pointers & Dynamic Allocation", topics: "Pointer arithmetic, dynamic arrays using malloc/free." },
            { unit: 5, title: "File Handling & Structures", topics: "Student record management system using binary files." }
          ],
          materials: [
            { unit: 1, title: "C Programming Lab Manual 2026-27", type: "PDF", size: "3.1 MB", date: "Aug 28, 2026", by: "Dr. Abhishek Sharma (AS)" }
          ]
        },
        {
          code: "33261107(A)",
          name: "Embedded Programming Lab",
          shortName: "Embedded Programming Lab",
          type: "Lab",
          credits: 1.5,
          faculty: "Dr. Rupam Shrivastava (RUS) / Dr. Nandkishor Joshi (NKJ) / Ms. Akanksha Bhatt (AB)",
          attended: 10,
          total: 12,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 9,
          classQuizMax: 10,
          centralQuiz: null,
          minorMarks: 27,
          minorStatus: "Evaluated (27/30)",
          majorMarks: null,
          majorStatus: "Starts Dec 05, 2026",
          units: [
            { unit: 1, title: "Microcontroller GPIO Interfacing", topics: "LED blinking, push button reading, digital input/output." },
            { unit: 2, title: "Sensor Integration (DHT22 & Ultrasonic)", topics: "Reading temperature, humidity, and distance measurements." },
            { unit: 3, title: "PWM Motor Control & Timers", topics: "DC motor speed control using PWM and timer interrupts." },
            { unit: 4, title: "UART & I2C Sensor Display", topics: "Serial monitor debugging and I2C OLED screen output." },
            { unit: 5, title: "Wi-Fi MQTT Cloud Telemetry", topics: "Publishing sensor telemetry to cloud broker." }
          ],
          materials: [
            { unit: 1, title: "Embedded Lab Kit & Experiment Guide", type: "PDF", size: "4.2 MB", date: "Aug 30, 2026", by: "Dr. Rupam Shrivastava (RUS)" }
          ]
        },
        {
          code: "33261110",
          name: "Language Lab",
          shortName: "Language Lab",
          type: "Lab",
          credits: 1.5,
          faculty: "Dr. Sanjeev Khanna (SK) / Dr. Arzoo Choubey (AC)",
          attended: 11,
          total: 12,
          simDeltaAttended: 0,
          simDeltaTotal: 0,
          classQuizMarks: 10,
          classQuizMax: 10,
          centralQuiz: null,
          minorMarks: 29,
          minorStatus: "Evaluated (29/30)",
          majorMarks: null,
          majorStatus: "Starts Dec 07, 2026",
          units: [
            { unit: 1, title: "Phonetics & English Pronunciation", topics: "Vowels, consonants, intonation patterns, and accent neutralization." },
            { unit: 2, title: "Technical Communication & Vocabulary", topics: "Professional writing, technical reports, and vocabulary building." },
            { unit: 3, title: "Group Discussions & Presentations", topics: "Public speaking, impromptu talks, and group dynamics." },
            { unit: 4, title: "Listening Comprehension & Audio Labs", topics: "Active listening, interactive exercises, and comprehension testing." },
            { unit: 5, title: "Interview Skills & Resume Building", topics: "Mock interviews, corporate etiquette, and professional profiling." }
          ],
          materials: [
            { unit: 1, title: "Language Lab Manual & Communication Guide", type: "PDF", size: "3.5 MB", date: "Sep 02, 2026", by: "Dr. Sanjeev Khanna (SK)" }
          ]
        }
      ],

      timetable: {
        Monday: [
          { time: "10:00 AM - 11:00 AM", code: "33261102", subject: "Computer Programming", room: "M003", faculty: "Dr. Abhishek Sharma (AS)", type: "Theory" },
          { time: "11:00 AM - 12:00 Noon", code: "33261101", subject: "Introduction to Electrical and Computer Engineering", room: "M003", faculty: "Dr. Saurabh Kumar Rajput (SKR)", type: "Theory" },
          { time: "01:00 PM - 02:00 PM", code: "LUNCH", subject: "Lunch Break", room: "CIOT Block", faculty: "—", type: "Break" },
          { time: "02:00 PM - 04:00 PM", code: "33261107(A)", subject: "Embedded Programming Lab (Batch A – 40 Students)", room: "M004", faculty: "Dr. Rupam Shrivastava (RUS) / Dr. Nandkishor Joshi (NKJ)", type: "Lab", batch: "Batch A" },
          { time: "02:00 PM - 04:00 PM", code: "33261106(B)", subject: "Computer Programming Lab (Batch B – 40 Students)", room: "M005", faculty: "Dr. Abhishek Sharma (AS) / Mr. Vinay Gupta (VG)", type: "Lab", batch: "Batch B" }
        ],
        Tuesday: [
          { time: "10:00 AM - 11:00 AM", code: "33261105", subject: "Linear Algebra and Calculus", room: "M003", faculty: "Dr. Nidhi Humnekar (NH)", type: "Theory", status: "Completed" },
          { time: "11:00 AM - 12:00 Noon", code: "33261104", subject: "Digital Logic Design", room: "M003", faculty: "Dr. Aftab Ahmed Ansari (AAA)", type: "Theory", status: "Completed" },
          { time: "12:00 Noon - 01:00 PM", code: "33261101", subject: "Introduction to Electrical and Computer Engineering (+ Live Quiz)", room: "M003", faculty: "Dr. Saurabh Kumar Rajput (SKR)", type: "Theory", status: "Ongoing" },
          { time: "01:00 PM Onwards", code: "BREAK", subject: "Break for Entire Day (No Afternoon Classes)", room: "—", faculty: "—", type: "Break" }
        ],
        Wednesday: [
          { time: "10:00 AM - 11:00 AM", code: "33261105", subject: "Linear Algebra and Calculus", room: "M003", faculty: "Dr. Nidhi Humnekar (NH)", type: "Theory" },
          { time: "11:00 AM - 01:00 PM", code: "33261109", subject: "Micro Project-I Lab (2 Hours)", room: "M003", faculty: "Dr. Abhishek Sharma (AS)", type: "Project" },
          { time: "01:00 PM - 02:00 PM", code: "LUNCH", subject: "Lunch Break", room: "CIOT Block", faculty: "—", type: "Break" },
          { time: "02:00 PM - 03:00 PM", code: "33261103", subject: "Basics of Embedded Programming", room: "M003", faculty: "Dr. Rupam Shrivastava (RUS)", type: "Theory" },
          { time: "03:00 PM - 04:00 PM", code: "33261104", subject: "Digital Logic Design", room: "M003", faculty: "Dr. Aftab Ahmed Ansari (AAA)", type: "Theory" },
          { time: "04:00 PM - 06:00 PM", code: "33261110", subject: "Language Lab (2 Hours)", room: "M003", faculty: "Dr. Sanjeev Khanna (SK) / Dr. Arzoo Choubey (AC)", type: "Lab" }
        ],
        Thursday: [
          { time: "09:00 AM - 11:00 AM", code: "33261106(A)", subject: "Computer Programming Lab (Batch A – 40 Students)", room: "M005", faculty: "Dr. Abhishek Sharma (AS) / Mr. Vinay Gupta (VG)", type: "Lab", batch: "Batch A" },
          { time: "09:00 AM - 11:00 AM", code: "33261107(B)", subject: "Embedded Programming Lab (Batch B – 40 Students)", room: "M004", faculty: "Dr. Rupam Shrivastava (RUS) / Ms. Akanksha Bhatt (AB)", type: "Lab", batch: "Batch B" },
          { time: "11:00 AM - 12:00 Noon", code: "33261103", subject: "Basics of Embedded Programming", room: "M003", faculty: "Dr. Rupam Shrivastava (RUS)", type: "Theory" },
          { time: "12:00 Noon - 01:00 PM", code: "33261102", subject: "Computer Programming", room: "M003", faculty: "Dr. Abhishek Sharma (AS)", type: "Theory" },
          { time: "01:00 PM - 02:00 PM", code: "LUNCH", subject: "Lunch Break", room: "CIOT Block", faculty: "—", type: "Break" }
        ],
        Friday: [
          { time: "09:00 AM - 11:00 AM", code: "33261108", subject: "Semester Proficiency", room: "M003", faculty: "Dr. Namita Arya (NA)", type: "Presentation" },
          { time: "11:00 AM - 12:00 Noon", code: "33261103", subject: "Basics of Embedded Programming", room: "M003", faculty: "Dr. Rupam Shrivastava (RUS)", type: "Theory" },
          { time: "12:00 Noon - 01:00 PM", code: "33261104", subject: "Digital Logic Design", room: "M003", faculty: "Dr. Aftab Ahmed Ansari (AAA)", type: "Theory" },
          { time: "01:00 PM - 02:00 PM", code: "LUNCH", subject: "Lunch Break", room: "CIOT Block", faculty: "—", type: "Break" },
          { time: "02:00 PM - 03:00 PM", code: "33261101", subject: "Introduction to Electrical and Computer Engineering", room: "M003", faculty: "Dr. Saurabh Kumar Rajput (SKR)", type: "Theory" },
          { time: "03:00 PM - 04:00 PM", code: "33261105", subject: "Linear Algebra and Calculus", room: "M003", faculty: "Dr. Nidhi Humnekar (NH)", type: "Theory" }
        ]
      },

      microProject: {
        courseCode: "33261109",
        track: "Software",
        supervisor: "Dr. Abhishek Sharma (AS)",
        approvalStatus: "Approved",
        title: "Smart Campus Energy & Attendance Edge Node",
        abstract: "An edge-assisted IoT and telemetry platform designed for CIOT classrooms at MITS Gwalior by Ashi Golhani. Combines ESP32 MQTT occupancy sensing with automated room HVAC power optimization and encrypted BLE attendance verification.",
        teamMembers: [
          "Ashi Golhani (BTEC26O1022) – Lead Developer",
          "Riya Kulshrestha (BTEC26O1041) – MQTT & Backend API",
          "Devansh Bhadoria (BTEC26O1018) – Edge Firmware & Testing"
        ],
        expoDates: "Nov 25–27, 2026 (CIOT Atrium & Seminar Hall)",
        milestones: [
          { id: 1, title: "Problem Statement, Literature Survey & Track Approval", deadline: "Sep 15, 2026", status: "Graded", score: "9.5 / 10", file: "M1_Synopsis_BTEC26O1022.pdf", remark: "Excellent scope tailored for CIOT smart classrooms." },
          { id: 2, title: "System Architecture, Flowchart & Component/API Design", deadline: "Oct 05, 2026", status: "Graded", score: "9.0 / 10", file: "M2_Architecture_Diagram.pdf", remark: "Clean MQTT topic hierarchy and low-latency state diagram." },
          { id: 3, title: "Core Prototype Development & Mid-Term Demonstration", deadline: "Oct 28, 2026", status: "Graded", score: "8.8 / 10", file: "M3_Prototype_Build_v1.zip", remark: "Dashboard telemetry working; optimize reconnection watchdog." },
          { id: 4, title: "Validation Testing, Power Profiling & Bug Fixes", deadline: "Nov 20, 2026", status: "Pending Upload", score: "Awaiting Review", file: null, remark: "Upload stress-test logs before Nov 20, 11:59 PM." },
          { id: 5, title: "Final Project Report, Poster & CIOT Expo Presentation", deadline: "Nov 25, 2026", status: "Upcoming", score: "Expo Evaluation", file: null, remark: "Evaluated by Dr. Abhishek Sharma (AS) & External Panel." }
        ]
      },

      scholarships: [
        {
          id: "SCH-MP-POST",
          name: "MP Post-Matric Scholarship (OBC / SC / ST)",
          authority: "Department of Backward Classes & Minorities Welfare, Govt. of M.P.",
          benefit: "Up to 100% Tuition Fee Reimbursement + Maintenance Allowance",
          deadline: "Dec 15, 2026",
          eligibility: "Domicile of Madhya Pradesh, Family Annual Income within prescribed slab, valid Samagra ID & Income Certificate.",
          status: "Institute Verified • DBT Pending",
          appId: "MPOBC269184"
        },
        {
          id: "SCH-MMVY",
          name: "Mukhyamantri Medhavi Chhatra Yojana (MMVY)",
          authority: "Directorate of Technical Education (DTE), Madhya Pradesh",
          benefit: "Full B.Tech Tuition Fee Coverage (₹1,40,000/yr) by State Govt.",
          deadline: "Dec 20, 2026",
          eligibility: "MP Board ≥ 70% or CBSE/ICSE ≥ 85% in Class 12th, JEE Main Rank < 1.5 Lakh, Annual Family Income < ₹8 Lakh.",
          status: "Open for Renewal / Fresh Filing",
          appId: "MMVY-2026-8841"
        },
        {
          id: "SCH-PRAGATI",
          name: "AICTE Pragati Scholarship for Girl Students",
          authority: "AICTE / National Scholarship Portal (NSP)",
          benefit: "₹50,000 per annum towards Tuition, Books & Computing Equipment",
          deadline: "Nov 30, 2026",
          eligibility: "Meritorious girl students admitted to 1st year of AICTE-approved technical degree course (up to 2 girls per family).",
          status: "Eligible • Documents Ready",
          appId: "NSP-PRG-26019"
        },
        {
          id: "SCH-CSS",
          name: "Central Sector Scheme of Scholarship (NSP)",
          authority: "Ministry of Education, Government of India",
          benefit: "₹12,000/yr (1st–3rd Yr) & ₹20,000/yr (4th Yr) Direct Benefit Transfer",
          deadline: "Nov 28, 2026",
          eligibility: "Top 20th percentile in Class XII board examination; regular B.Tech enrollment.",
          status: "Open on NSP Portal",
          appId: "NSP-CSS-2026"
        }
      ],

      tickets: [
        { id: "TKT-2026-418", category: "Academic", subject: "Lab Batch A Roll List Verification (40 Students Cohort)", date: "Nov 11, 2026", status: "Resolved", reply: "Verified by CIOT Office. Your name (Ashi Golhani) is confirmed under Batch A (40 students) for Labs." },
        { id: "TKT-2026-452", category: "Scholarship", subject: "Bonafide & Fee Structure Attestation for MP Post-Matric / MMVY", date: "Nov 15, 2026", status: "In Progress", reply: "Digitally signed Bonafide enabled under the Bonafide & Official Letters tab." }
      ],

      clubsDirectory: [
        { name: "Bandish (Music Club)", category: "Cultural & Literary", lead: "Kabir Shrivastava", faculty: "Dr. Namita Arya", desc: "Official vocal, classical, and instrumental music society of MITS Gwalior." },
        { name: "Nitya Sangam (Dance Club)", category: "Cultural & Literary", lead: "Ishita Tomar", faculty: "Prof. Meenakshi Mazumdar", desc: "Classical, contemporary, and street dance troupe representing MITS at national fests." },
        { name: "Hindi Samiti", category: "Cultural & Literary", lead: "Prashant Dubey", faculty: "Dr. Sanjeev Khanna", desc: "Promotes Hindi literature, Kavi Sammelan, debates, and cultural heritage." },
        { name: "MITS ACM Student Chapter", category: "Tech & Societies", lead: "Ananya Deshmukh", faculty: "Dr. Abhishek Sharma", desc: "Premier computing society hosting hackathons, algorithmic contests, and research talks." },
        { name: "GeeksforGeeks MITS-DU", category: "Tech & Societies", lead: "Rohan Patidar", faculty: "Prof. Sneha Verma", desc: "DSA bootcamps, competitive coding leagues, and placement prep community." },
        { name: "AI Club", category: "Tech & Societies", lead: "Siddharth Jain", faculty: "Dr. Praveen Bansal", desc: "Deep learning, generative AI workshops, and neural architecture reading groups." },
        { name: "Electronics Club", category: "Tech & Societies", lead: "Harshvardhan Joshi", faculty: "Dr. Abhishek Sharma", desc: "PCB design, VLSI, robotics, and embedded hardware prototyping." },
        { name: "Digital Learning Group (DLG)", category: "Tech & Societies", lead: "Tanvi Agrawal", faculty: "Dr. Nand Kishore Joshi", desc: "UI/UX design, full-stack web engineering, and open-source digital initiatives." },
        { name: "GSX Gwalior", category: "Tech & Societies", lead: "Yashraj Chauhan", faculty: "Dr. Praveen Bansal", desc: "Git, open-source contribution sprints, cloud DevOps, and developer community." },
        { name: "IoT Society", category: "Tech & Societies", lead: "Nikhil Bansal", faculty: "Dr. Praveen Bansal", desc: "Flagship student society of CIOT building smart campus and IIoT deployments." },
        { name: "IEEE Student Branch", category: "Tech & Societies", lead: "Shreya Kulkarni", faculty: "Dr. R.K. Tiwari", desc: "Global technical standards, IEEE paper publishing, and interdisciplinary symposiums." },
        { name: "Research & ML Club", category: "Tech & Societies", lead: "Aditya Dixit", faculty: "Dr. Nand Kishore Joshi", desc: "Undergraduate research mentorship, patent filing, and machine learning labs." },
        { name: "Finance Club", category: "Professional, Sports & Cells", lead: "Manan Gupta", faculty: "Dr. Vikas Shinde", desc: "Financial literacy, mock stock trading, fintech case studies, and economics." },
        { name: "MITS Sports / Badminton Club", category: "Professional, Sports & Cells", lead: "Vikramaditya Rana", faculty: "Dr. B.P.S. Bhadoria", desc: "Inter-branch tournaments, indoor badminton arena, and university athletics." },
        { name: "Girls Grievance & Gender Sensitization Cell (GGC)", category: "Professional, Sports & Cells", lead: "Dr. Namita Arya (Chair)", faculty: "Dr. Namita Arya", desc: "Dedicated cell ensuring campus safety, equity, wellness, and gender sensitization." }
      ],

      events: [
        {
          id: "EVT-101",
          title: "CodeRush 3.0",
          club: "MITS ACM Student Chapter",
          category: "Tech & Societies",
          date: "Nov 21, 2026 • 10:00 AM",
          venue: "SH-3 & CIOT Computing Labs",
          fees: { solo: "₹99", duo: "₹179", squad: "₹299 (Team of 4)" },
          perks: "Hard Copy Merit Certificates + E-Certificates + ₹15,000 Prize Pool",
          desc: "Flagship 4-hour competitive programming & algorithmic debugging championship for 1st to 4th year coders.",
          registrations: 184,
          registered: false
        },
        {
          id: "EVT-102",
          title: "TechPulse 2026",
          club: "IoT Society & Electronics Club",
          category: "Tech & Societies",
          date: "Nov 23, 2026 • 11:00 AM",
          venue: "SH-5 (Seminar Hall 5)",
          fees: { solo: "Free", duo: "₹100", squad: "₹180 (Team of 3)" },
          perks: "E-Certificates + Hardware Starter Kits for Top 3 Teams",
          desc: "Hands-on sensor fusion workshop & rapid hardware ideation challenge hosted at CIOT.",
          registrations: 142,
          registered: false
        }
      ],

      facultyRoster: [
        { roll: "BTEC26O1004", name: "Aarav Sharma", batch: "Batch A", present: true },
        { roll: "BTEC26O1011", name: "Anushka Agrawal", batch: "Batch A", present: true },
        { roll: "BTEC26O1018", name: "Devansh Bhadoria", batch: "Batch A", present: true },
        { roll: "BTEC26O1022", name: "Ashi Golhani", batch: "Batch A", present: true },
        { roll: "BTEC26O1029", name: "Harshit Rathore", batch: "Batch A", present: false },
        { roll: "BTEC26O1035", name: "Kritika Shrivastava", batch: "Batch A", present: true },
        { roll: "BTEC26O1041", name: "Riya Kulshrestha", batch: "Batch A", present: true },
        { roll: "BTEC26O1048", name: "Samarth Tiwari", batch: "Batch A", present: false },
        { roll: "BTEC26O1053", name: "Nandini Chauhan", batch: "Batch B", present: true },
        { roll: "BTEC26O1061", name: "Pranav Dixit", batch: "Batch B", present: true },
        { roll: "BTEC26O1069", name: "Shruti Tomar", batch: "Batch B", present: true },
        { roll: "BTEC26O1076", name: "Yashvardhan Singh", batch: "Batch B", present: false }
      ],

      quizzes: [
        {
          id: "QZ-33261101-4",
          title: "In-Class Live Quiz #4: Intro to ECE & Circuit Fundamentals",
          courseCode: "33261101",
          courseName: "Introduction to Electrical and Computer Engineering",
          faculty: "Dr. Saurabh Kumar Rajput (SKR)",
          durationMins: 10,
          totalMarks: 13,
          status: "not_started",
          strikes: 0,
          timeLeft: 600,
          score: 0,
          grade: "A+",
          percentile: "96.2%ile",
          answers: { q1: null, q2: [], q3: "", q4: { A: "", B: "", C: "" } },
          questions: [
            {
              id: 1,
              type: "mcq",
              points: 2,
              text: "In a balanced three-phase star-connected electrical system with line voltage 415V, what is the approximate phase voltage?",
              options: ["240V", "415V", "120V", "1200V"],
              correctIndex: 0
            },
            {
              id: 2,
              type: "checkbox",
              points: 4,
              text: "Select all valid semiconductor devices and diodes used in power conditioning circuits:",
              options: ["PN Junction Diode", "Zener Voltage Regulator", "Resistive Voltage Divider", "MOSFET Power Switch"],
              correctIndices: [0, 1, 3]
            },
            {
              id: 3,
              type: "integer",
              points: 3,
              text: "Calculate the total equivalent resistance (in Ohms) of three 12Ω resistors connected in parallel:",
              correct: 4
            },
            {
              id: 4,
              type: "match",
              points: 4,
              text: "Match the following EACE circuit principles with their correct definitions:",
              left: [
                { key: "A", label: "Kirchhoff's Current Law (KCL)", match: "Conservation of Electric Charge at a node" },
                { key: "B", label: "Kirchhoff's Voltage Law (KVL)", match: "Conservation of Energy around a closed loop" },
                { key: "C", label: "Quality Factor (Q-Factor)", match: "Ratio of resonant frequency to bandwidth (ω0 / BW)" }
              ],
              rightOptions: [
                "Conservation of Electric Charge at a node",
                "Conservation of Energy around a closed loop",
                "Ratio of resonant frequency to bandwidth (ω0 / BW)"
              ]
            }
          ]
        }
      ],

      facultyQuizDraftQuestions: [
        {
          type: "mcq",
          points: 2,
          text: "Which C standard library function dynamically allocates memory and initializes all bytes to zero?",
          options: ["malloc()", "calloc()", "realloc()", "free()"],
          correctIndex: 1
        },
        {
          type: "integer",
          points: 3,
          text: "What is the output of sizeof(int) in bytes on a standard 32/64-bit GCC x86 compiler?",
          correct: 4
        }
      ]
    };

    function loadPersistedState() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (!raw) return structuredClone(DEFAULT_STATE);
        const parsed = JSON.parse(raw);
        return {
          ...structuredClone(DEFAULT_STATE),
          ...parsed,
          studentProfile: { ...DEFAULT_STATE.studentProfile, ...(parsed.studentProfile || {}) },
          facultyProfile: { ...DEFAULT_STATE.facultyProfile, ...(parsed.facultyProfile || {}) },
          clubProfile: { ...DEFAULT_STATE.clubProfile, ...(parsed.clubProfile || {}) },
          timetable: DEFAULT_STATE.timetable,
          scholarships: DEFAULT_STATE.scholarships
        };
      } catch (e) {
        return structuredClone(DEFAULT_STATE);
      }
    }

    let state = loadPersistedState();

    function saveState() {
      try {
        const toSave = {
          currentPortal: state.currentPortal,
          studentTab: state.studentTab,
          facultyTab: state.facultyTab,
          clubTab: state.clubTab,
          selectedDay: state.selectedDay,
          selectedBatch: state.selectedBatch,
          selectedCourseCode: state.selectedCourseCode,
          clubCategoryFilter: state.clubCategoryFilter,
          servicesSubTab: state.servicesSubTab,
          feeSubTab: state.feeSubTab,
          facultyCohortMode: state.facultyCohortMode,
          facultyMarksTier: state.facultyMarksTier,
          activeQuizIndex: state.activeQuizIndex,
          announcements: state.announcements,
          studentProfile: state.studentProfile,
          facultyProfile: state.facultyProfile,
          clubProfile: state.clubProfile,
          courses: state.courses,
          microProject: state.microProject,
          tickets: state.tickets,
          events: state.events,
          facultyRoster: state.facultyRoster,
          quizzes: state.quizzes.map(q => ({
            ...q,
            status: q.status === 'in_progress' ? 'not_started' : q.status
          })),
          facultyQuizDraftQuestions: state.facultyQuizDraftQuestions
        };
        localStorage.setItem(STORAGE_KEY, JSON.stringify(toSave));
      } catch (e) {}
    }

    let quizInterval = null;
    let gradingChartInstance = null;

    function showToast(message, type = 'info') {
      const container = document.getElementById('toast-container');
      if (!container) return;
      const colors = {
        success: 'bg-emerald-900 text-white border-emerald-500',
        warning: 'bg-amber-900 text-white border-amber-500',
        danger: 'bg-rose-900 text-white border-rose-500',
        info: 'bg-mits-navy text-white border-amber-400'
      };
      const icons = {
        success: 'fa-circle-check text-emerald-400',
        warning: 'fa-triangle-exclamation text-amber-400',
        danger: 'fa-circle-xmark text-rose-400',
        info: 'fa-circle-info text-amber-400'
      };
      const toast = document.createElement('div');
      toast.className = `pointer-events-auto flex items-start gap-3 p-3.5 rounded-xl shadow-xl border-l-4 ${colors[type] || colors.info} text-sm transition-all duration-300 transform translate-y-2 opacity-0`;
      toast.innerHTML = `
        <i class="fa-solid ${icons[type] || icons.info} mt-0.5 text-base"></i>
        <div class="flex-1 font-medium leading-snug">${message}</div>
      `;
      container.appendChild(toast);
      requestAnimationFrame(() => {
        toast.classList.remove('translate-y-2', 'opacity-0');
      });
      setTimeout(() => {
        toast.classList.add('opacity-0', 'translate-y-2');
        setTimeout(() => toast.remove(), 300);
      }, 3800);
    }

    function openModal(htmlContent) {
      const backdrop = document.getElementById('modal-backdrop');
      const content = document.getElementById('modal-content');
      content.innerHTML = htmlContent;
      backdrop.classList.remove('hidden');
      backdrop.classList.add('flex');
    }

    function closeModal() {
      const backdrop = document.getElementById('modal-backdrop');
      backdrop.classList.add('hidden');
      backdrop.classList.remove('flex');
    }

    function getAttendanceStats(course) {
      const att = course.attended + (course.simDeltaAttended || 0);
      const tot = Math.max(1, course.total + (course.simDeltaTotal || 0));
      const pct = (att / tot) * 100;
      const isSafe = pct >= 75.0;

      let predictorText = "";
      let count = 0;
      if (isSafe) {
        count = Math.floor((att - 0.75 * tot) / 0.75);
        predictorText = count > 0
          ? `Safe to bunk next <strong class="text-emerald-700">${count}</strong> consecutive class${count > 1 ? 'es' : ''}`
          : `Borderline Safe — Do <strong class="text-amber-700">not</strong> miss the next class`;
      } else {
        count = Math.ceil((0.75 * tot - att) / 0.25);
        predictorText = `Attend next <strong class="text-rose-700">${count}</strong> consecutive class${count > 1 ? 'es' : ''} to reach 75%`;
      }

      return { att, tot, pct, isSafe, count, predictorText };
    }

    function getAggregateAttendance() {
      let totalAtt = 0;
      let totalHeld = 0;
      state.courses.forEach(c => {
        totalAtt += c.attended;
        totalHeld += c.total;
      });
      return ((totalAtt / totalHeld) * 100).toFixed(1);
    }

    function getMitsCrestSVG(size = "w-11 h-11") {
      return `
        <svg class="${size} shrink-0" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="32" cy="32" r="30" fill="#0B1D3A" stroke="#F59E0B" stroke-width="2.5"/>
          <circle cx="32" cy="32" r="24" stroke="#FBBF24" stroke-width="1" stroke-dasharray="2 2"/>
          <path d="M32 12L46 19V33C46 42.5 39.8 49.8 32 52C24.2 49.8 18 42.5 18 33V19L32 12Z" fill="#1E3A8A" stroke="#F59E0B" stroke-width="1.8"/>
          <circle cx="32" cy="27" r="4.5" fill="#FBBF24"/>
          <path d="M23 39C25.5 35 28.5 33.5 32 33.5C35.5 33.5 38.5 35 41 39" stroke="#FBBF24" stroke-width="2" stroke-linecap="round"/>
          <path d="M26 21L32 17L38 21" stroke="#FDE68A" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
      `;
    }

    function loginAs(portal) {
      if (portal === 'student') {
        const idVal = document.getElementById('student-id-input')?.value?.trim();
        if (idVal) {
          if (idVal.includes('@')) {
            state.studentProfile.email = idVal;
          } else {
            state.studentProfile.enrollment = idVal.toUpperCase();
          }
        }
        state.currentPortal = 'student';
        state.studentTab = 'dashboard';
        showToast(`Welcome back, ${state.studentProfile.name} (${state.studentProfile.enrollment})!`, 'success');
      } else if (portal === 'faculty') {
        const facEmail = document.getElementById('faculty-email-input')?.value?.trim();
        if (facEmail && facEmail.includes('praveen')) {
          state.facultyProfile.name = "Dr. Praveen Bansal";
          state.facultyProfile.designation = "Professor & Dean, Centre for IoT";
          state.facultyProfile.email = facEmail;
        } else {
          state.facultyProfile.name = "Dr. Abhishek Sharma (AS)";
          state.facultyProfile.designation = "Associate Professor & Branch Coordinator (EACE)";
          state.facultyProfile.email = facEmail || "abhishek.sharma@mitsgwl.ac.in";
        }
        state.currentPortal = 'faculty';
        state.facultyTab = 'attendance';
        showToast(`Logged in to Faculty Portal as ${state.facultyProfile.name}`, 'success');
      } else if (portal === 'club') {
        const clubEmail = document.getElementById('club-email-input')?.value?.trim();
        const clubRole = document.getElementById('club-role-select')?.value || "President";
        if (clubEmail) state.clubProfile.email = clubEmail;
        state.clubProfile.role = clubRole;
        state.currentPortal = 'club';
        state.clubTab = 'create-event';
        showToast(`Authenticated as ${clubRole} — ${state.clubProfile.clubName}`, 'success');
      }
      render();
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function logoutToLanding() {
      if (quizInterval) {
        clearInterval(quizInterval);
        quizInterval = null;
      }
      state.currentPortal = 'landing';
      state.sidebarOpen = false;
      render();
      showToast("Signed out of SAMS session.", "info");
    }

    function renderLandingPage() {
      return `
        <div class="bg-mits-navy text-slate-200 text-xs py-2 px-4 border-b border-amber-500/30">
          <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-2">
            <div class="flex items-center gap-2">
              <span class="inline-block w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
              <span class="font-medium text-amber-300">Academic Session: July – Dec 2026 (w.e.f. 24.08.2026)</span>
              <span class="hidden sm:inline text-slate-400">|</span>
              <span class="hidden sm:inline">NAAC Accredited with A++ Grade • Deemed University</span>
            </div>
            <div class="flex items-center gap-4 font-mono text-[11px]">
              <span><i class="fa-regular fa-calendar mr-1 text-amber-400"></i>${CURRENT_DATE_STR}</span>
              <span class="hidden md:inline text-amber-300">CIOT Block (M003–M005)</span>
            </div>
          </div>
        </div>

        <div class="bg-amber-50 border-b border-amber-200 py-2 overflow-hidden flex items-center">
          <div class="bg-mits-gold text-white text-[11px] font-bold uppercase tracking-wider px-3 py-1 rounded-r-full shrink-0 z-10 shadow-xs flex items-center gap-1.5">
            <i class="fa-solid fa-bullhorn"></i> CIOT Live Ticker
          </div>
          <div class="overflow-hidden flex-1 ml-3">
            <div class="animate-ticker text-xs font-medium text-slate-800 space-x-12">
              ${state.announcements.map(a => `<span>${a}</span>`).join('')}
              ${state.announcements.map(a => `<span>${a}</span>`).join('')}
            </div>
          </div>
        </div>

        <header class="bg-gradient-to-br from-mits-navy via-mits-royal to-indigo-950 text-white relative overflow-hidden">
          <div class="absolute inset-0 opacity-10 pointer-events-none" style="background-image: radial-gradient(#F59E0B 1px, transparent 1px); background-size: 24px 24px;"></div>
          <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-10 md:py-14 relative z-10">
            <div class="flex flex-col lg:flex-row items-center justify-between gap-8">
              <div class="flex flex-col sm:flex-row items-center sm:items-start text-center sm:text-left gap-5 max-w-3xl">
                ${getMitsCrestSVG("w-20 h-20 sm:w-24 sm:h-24 drop-shadow-lg")}
                <div>
                  <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-amber-500/20 border border-amber-400/40 text-amber-300 text-xs font-semibold tracking-wide mb-3">
                    <i class="fa-solid fa-microchip"></i> CENTRE FOR INTERNET OF THINGS (CIOT)
                  </div>
                  <h1 class="text-2xl sm:text-4xl font-extrabold tracking-tight leading-tight">
                    Madhav Institute of Technology & Science
                  </h1>
                  <p class="text-amber-300 font-semibold text-sm sm:text-base mt-1">
                    Deemed University, Gwalior (M.P.), INDIA • Est. 1957
                  </p>
                  <p class="text-slate-300 text-sm mt-3 max-w-2xl leading-relaxed">
                    Welcome to <strong class="text-white">SAMS (Student Academic Management System)</strong> — unified digital academic ERP for real-time 75% attendance analytics, live proctored quizzes, 4-tier exam scorebooks, Micro-Project-I milestones, and campus club ecosystems.
                  </p>
                </div>
              </div>

              <div class="grid grid-cols-2 gap-3 w-full lg:w-auto shrink-0">
                <div class="bg-white/10 backdrop-blur-md border border-white/15 rounded-2xl p-4 text-center">
                  <div class="text-2xl font-extrabold text-amber-400">CIOT</div>
                  <div class="text-xs text-slate-300 mt-0.5">EACE Dept. (80)</div>
                </div>
                <div class="bg-white/10 backdrop-blur-md border border-white/15 rounded-2xl p-4 text-center">
                  <div class="text-2xl font-extrabold text-emerald-400">5-Tier</div>
                  <div class="text-xs text-slate-300 mt-0.5">NEP Assessment</div>
                </div>
                <div class="bg-white/10 backdrop-blur-md border border-white/15 rounded-2xl p-4 text-center">
                  <div class="text-2xl font-extrabold text-sky-300">75%</div>
                  <div class="text-xs text-slate-300 mt-0.5">Smart Bunk Engine</div>
                </div>
                <div class="bg-white/10 backdrop-blur-md border border-white/15 rounded-2xl p-4 text-center">
                  <div class="text-2xl font-extrabold text-amber-300">15+</div>
                  <div class="text-xs text-slate-300 mt-0.5">Active MITS Clubs</div>
                </div>
              </div>
            </div>
          </div>
        </header>

        <main class="flex-1 max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 -mt-6 pb-16 relative z-20 w-full">
          <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <div class="bg-white rounded-2xl shadow-xl border border-slate-200 overflow-hidden flex flex-col hover:border-indigo-400 transition-all">
              <div class="bg-gradient-to-r from-indigo-900 to-blue-800 p-5 text-white flex items-center justify-between">
                <div>
                  <span class="text-[11px] font-bold uppercase tracking-wider text-amber-300">Gateway 01</span>
                  <h2 class="text-xl font-bold mt-0.5 flex items-center gap-2">
                    <i class="fa-solid fa-user-graduate text-amber-400"></i> Student Portal
                  </h2>
                </div>
                <span class="px-2.5 py-1 rounded-lg bg-white/15 text-xs font-mono">B.Tech 2026</span>
              </div>
              <div class="p-6 flex-1 flex flex-col justify-between space-y-5">
                <div class="space-y-4">
                  <p class="text-xs text-slate-500 leading-relaxed">
                    Access 1st Sem EACE dashboard, official timetable, 75% attendance predictor, live anti-cheating quizzes, 5-tier exam scorebook & scholarships.
                  </p>
                  <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Enrollment Number or College Email</label>
                    <div class="relative">
                      <i class="fa-solid fa-id-card absolute left-3.5 top-3 text-slate-400 text-sm"></i>
                      <input id="student-id-input" type="text" value="BTEC26O1022" placeholder="e.g. BTEC26O1022 or btec26o1022@mitsgwl.ac.in"
                        class="w-full pl-10 pr-3 py-2.5 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 font-mono" />
                    </div>
                  </div>
                  <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">SAMS Password</label>
                    <div class="relative">
                      <i class="fa-solid fa-lock absolute left-3.5 top-3 text-slate-400 text-sm"></i>
                      <input type="password" value="••••••••••••"
                        class="w-full pl-10 pr-3 py-2.5 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" />
                    </div>
                  </div>
                </div>
                <div class="space-y-2.5 pt-2">
                  <button onclick="loginAs('student')"
                    class="w-full py-3 px-4 bg-indigo-900 hover:bg-indigo-800 text-white font-semibold rounded-xl shadow-md transition flex items-center justify-center gap-2 cursor-pointer">
                    <span>Sign In as Student</span>
                    <i class="fa-solid fa-arrow-right"></i>
                  </button>
                  <button onclick="loginAs('student')"
                    class="w-full py-2 px-3 bg-indigo-50 hover:bg-indigo-100 text-indigo-900 text-xs font-semibold rounded-xl border border-indigo-200 transition flex items-center justify-center gap-1.5 cursor-pointer">
                    <i class="fa-solid fa-bolt text-amber-500"></i> Demo Quick Login: Ashi Golhani (BTEC26O1022)
                  </button>
                </div>
              </div>
            </div>

            <div class="bg-white rounded-2xl shadow-xl border border-slate-200 overflow-hidden flex flex-col hover:border-amber-500 transition-all">
              <div class="bg-gradient-to-r from-mits-navy to-slate-800 p-5 text-white flex items-center justify-between">
                <div>
                  <span class="text-[11px] font-bold uppercase tracking-wider text-amber-400">Gateway 02</span>
                  <h2 class="text-xl font-bold mt-0.5 flex items-center gap-2">
                    <i class="fa-solid fa-chalkboard-user text-amber-400"></i> Faculty Portal
                  </h2>
                </div>
                <span class="px-2.5 py-1 rounded-lg bg-white/15 text-xs font-mono">CIOT ERP</span>
              </div>
              <div class="p-6 flex-1 flex flex-col justify-between space-y-5">
                <div class="space-y-4">
                  <p class="text-xs text-slate-500 leading-relaxed">
                    Mark EACE attendance (80-student theory cohort or Batch A/B labs), create & publish live quizzes to students, upload unit PDFs & enter 4-tier exam marks.
                  </p>
                  <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Official MITS Faculty Gmail ID</label>
                    <div class="relative">
                      <i class="fa-solid fa-envelope absolute left-3.5 top-3 text-slate-400 text-sm"></i>
                      <input id="faculty-email-input" type="email" value="abhishek.sharma@mitsgwl.ac.in" placeholder="faculty@mitsgwl.ac.in"
                        class="w-full pl-10 pr-3 py-2.5 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-amber-500 focus:border-amber-500 font-mono" />
                    </div>
                  </div>
                  <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">SSO / ERP Password</label>
                    <div class="relative">
                      <i class="fa-solid fa-key absolute left-3.5 top-3 text-slate-400 text-sm"></i>
                      <input type="password" value="••••••••••••"
                        class="w-full pl-10 pr-3 py-2.5 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-amber-500 focus:border-amber-500" />
                    </div>
                  </div>
                </div>
                <div class="space-y-2.5 pt-2">
                  <button onclick="loginAs('faculty')"
                    class="w-full py-3 px-4 bg-mits-navy hover:bg-slate-800 text-white font-semibold rounded-xl shadow-md transition flex items-center justify-center gap-2 cursor-pointer">
                    <span>Sign In as Faculty</span>
                    <i class="fa-solid fa-arrow-right"></i>
                  </button>
                  <button onclick="loginAs('faculty')"
                    class="w-full py-2 px-3 bg-amber-50 hover:bg-amber-100 text-amber-900 text-xs font-semibold rounded-xl border border-amber-200 transition flex items-center justify-center gap-1.5 cursor-pointer">
                    <i class="fa-solid fa-bolt text-amber-600"></i> Demo Quick Login: Dr. Abhishek Sharma (AS)
                  </button>
                </div>
              </div>
            </div>

            <div class="bg-white rounded-2xl shadow-xl border border-slate-200 overflow-hidden flex flex-col hover:border-emerald-500 transition-all">
              <div class="bg-gradient-to-r from-emerald-900 to-teal-800 p-5 text-white flex items-center justify-between">
                <div>
                  <span class="text-[11px] font-bold uppercase tracking-wider text-emerald-300">Gateway 03</span>
                  <h2 class="text-xl font-bold mt-0.5 flex items-center gap-2">
                    <i class="fa-solid fa-users-gear text-emerald-300"></i> Club & Society Portal
                  </h2>
                </div>
                <span class="px-2.5 py-1 rounded-lg bg-white/15 text-xs font-mono">SAC Hub</span>
              </div>
              <div class="p-6 flex-1 flex flex-col justify-between space-y-5">
                <div class="space-y-3.5">
                  <p class="text-xs text-slate-500 leading-relaxed">
                    Authorized access for Club Supervisors, Presidents & Vice Presidents to publish workshops, competitions & campus announcements.
                  </p>
                  <div class="grid grid-cols-1 sm:grid-cols-2 gap-2.5">
                    <div>
                      <label class="block text-xs font-semibold text-slate-700 mb-1">Office Bearer Role</label>
                      <select id="club-role-select" class="w-full px-3 py-2 text-xs font-semibold border border-slate-300 rounded-xl bg-white focus:ring-2 focus:ring-emerald-500">
                        <option value="President">President</option>
                        <option value="Vice President">Vice President</option>
                        <option value="Faculty Supervisor">Faculty Supervisor</option>
                      </select>
                    </div>
                    <div>
                      <label class="block text-xs font-semibold text-slate-700 mb-1">Club Gmail ID</label>
                      <input id="club-email-input" type="email" value="president.acm@mitsgwl.ac.in"
                        class="w-full px-3 py-2 text-xs border border-slate-300 rounded-xl font-mono focus:ring-2 focus:ring-emerald-500" />
                    </div>
                  </div>
                  <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Club Council Passkey</label>
                    <input type="password" value="••••••••••"
                      class="w-full px-3.5 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500" />
                  </div>
                </div>
                <div class="space-y-2.5 pt-2">
                  <button onclick="loginAs('club')"
                    class="w-full py-3 px-4 bg-emerald-800 hover:bg-emerald-700 text-white font-semibold rounded-xl shadow-md transition flex items-center justify-center gap-2 cursor-pointer">
                    <span>Sign In to Club Portal</span>
                    <i class="fa-solid fa-arrow-right"></i>
                  </button>
                  <button onclick="loginAs('club')"
                    class="w-full py-2 px-3 bg-emerald-50 hover:bg-emerald-100 text-emerald-900 text-xs font-semibold rounded-xl border border-emerald-200 transition flex items-center justify-center gap-1.5 cursor-pointer">
                    <i class="fa-solid fa-bolt text-emerald-600"></i> Demo Quick Login: MITS ACM President
                  </button>
                </div>
              </div>
            </div>

          </div>

          <div class="mt-10 bg-white rounded-2xl border border-slate-200 p-6 shadow-xs flex flex-col md:flex-row items-center justify-between gap-6">
            <div class="flex items-center gap-4">
              <div class="w-12 h-12 rounded-2xl bg-amber-100 text-amber-700 flex items-center justify-center text-xl shrink-0">
                <i class="fa-solid fa-building-columns"></i>
              </div>
              <div>
                <h3 class="font-bold text-slate-900 text-sm sm:text-base">Centre for Internet of Things (CIOT) — Department Leadership</h3>
                <p class="text-xs text-slate-600 mt-0.5">
                  <strong>Dean, Centre for IoT:</strong> Dr. Praveen Bansal &nbsp;•&nbsp;
                  <strong>Branch Coordinators (EACE):</strong> Dr. Abhishek Sharma & Dr. Nand Kishore Joshi
                </p>
              </div>
            </div>
            <div class="flex flex-wrap items-center gap-2 text-xs">
              <span class="px-3 py-1.5 rounded-full bg-slate-100 text-slate-700 font-medium"><i class="fa-solid fa-shield-halved text-indigo-600 mr-1.5"></i>2-Strike Anti-Cheat Engine</span>
              <span class="px-3 py-1.5 rounded-full bg-slate-100 text-slate-700 font-medium"><i class="fa-solid fa-arrows-rotate text-emerald-600 mr-1.5"></i>Cross-Portal Live Sync</span>
            </div>
          </div>
        </main>
      `;
    }

    function setStudentTab(tab) {
      state.studentTab = tab;
      state.sidebarOpen = false;
      render();
      const activeQz = state.quizzes[state.activeQuizIndex] || state.quizzes[0];
      if (tab === 'quiz' && activeQz && activeQz.status === 'submitted') {
        setTimeout(renderGradingBellCurve, 50);
      }
    }

    function renderStudentPortal() {
      const p = state.studentProfile;
      const aggAtt = getAggregateAttendance();
      const dangerCount = state.courses.filter(c => getAttendanceStats(c).pct < 75).length;
      const unattemptedQuizzes = state.quizzes.filter(q => q.status === 'not_started').length;

      const navItems = [
        { id: 'dashboard', label: 'Dashboard & Schedule', icon: 'fa-gauge-high' },
        { id: 'attendance', label: 'Smart Attendance (75%)', icon: 'fa-chart-pie', badge: dangerCount > 0 ? `${dangerCount} Alert` : 'Safe', badgeColor: dangerCount > 0 ? 'bg-rose-500 text-white' : 'bg-emerald-500 text-white' },
        { id: 'syllabus', label: 'Syllabus & Study Notes', icon: 'fa-book-open' },
        { id: 'quiz', label: 'Live In-Class Quizzes', icon: 'fa-stopwatch', badge: unattemptedQuizzes > 0 ? `${unattemptedQuizzes} LIVE` : 'Done', badgeColor: unattemptedQuizzes > 0 ? 'bg-amber-400 text-slate-900 animate-pulse' : 'bg-emerald-500 text-white' },
        { id: 'exams', label: 'Exams & 4-Tier Scorebook', icon: 'fa-file-signature' },
        { id: 'microproject', label: 'Micro-Project-I & Prof.', icon: 'fa-diagram-project' },
        { id: 'services', label: 'Fees, Scholarships & Desk', icon: 'fa-hand-holding-dollar' },
        { id: 'clubs', label: 'Clubs & Events Hub', icon: 'fa-calendar-days', badge: `${state.events.length}`, badgeColor: 'bg-indigo-500 text-white' },
        { id: 'profile', label: 'My Profile', icon: 'fa-user-circle' }
      ];

      return `
        <div class="min-h-screen flex bg-slate-100">
          ${state.sidebarOpen ? `<div onclick="state.sidebarOpen=false; render();" class="fixed inset-0 bg-slate-900/50 z-30 lg:hidden"></div>` : ''}

          <aside class="fixed lg:sticky top-0 left-0 z-40 h-screen w-72 bg-mits-navy text-white flex flex-col transition-transform duration-300 ${state.sidebarOpen ? 'translate-x-0' : '-translate-x-full lg:translate-x-0'} shrink-0 shadow-2xl">
            <div class="p-4 border-b border-white/10 flex items-center justify-between">
              <div class="flex items-center gap-3">
                ${getMitsCrestSVG("w-10 h-10")}
                <div>
                  <div class="font-extrabold text-base tracking-tight leading-none">MITS SAMS</div>
                  <div class="text-[11px] text-amber-400 font-semibold mt-1">Centre for IoT (CIOT)</div>
                </div>
              </div>
              <button onclick="state.sidebarOpen=false; render();" class="lg:hidden text-slate-300 hover:text-white p-1.5">
                <i class="fa-solid fa-xmark text-lg"></i>
              </button>
            </div>

            <div onclick="setStudentTab('profile')" class="p-4 bg-white/5 hover:bg-white/10 border-b border-white/10 cursor-pointer transition">
              <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-amber-400 to-amber-600 text-mits-navy font-extrabold flex items-center justify-center text-sm shadow">
                  AG
                </div>
                <div class="min-w-0">
                  <div class="text-sm font-bold truncate">${p.name}</div>
                  <div class="text-xs font-mono text-amber-300">${p.enrollment}</div>
                  <div class="text-[11px] text-slate-400">EACE • 1st Sem • ${p.batch}</div>
                </div>
              </div>
            </div>

            <nav class="flex-1 overflow-y-auto p-3 space-y-1">
              ${navItems.map(item => {
                const active = state.studentTab === item.id;
                return `
                  <button onclick="setStudentTab('${item.id}')"
                    class="w-full flex items-center justify-between px-3.5 py-2.5 rounded-xl text-xs font-semibold transition cursor-pointer ${active ? 'bg-amber-500 text-mits-navy shadow-md' : 'text-slate-300 hover:bg-white/10 hover:text-white'}">
                    <span class="flex items-center gap-3">
                      <i class="fa-solid ${item.icon} w-4 text-center ${active ? 'text-mits-navy' : 'text-amber-400'}"></i>
                      <span>${item.label}</span>
                    </span>
                    ${item.badge ? `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold ${active ? 'bg-mits-navy text-amber-300' : item.badgeColor}">${item.badge}</span>` : ''}
                  </button>
                `;
              }).join('')}
            </nav>

            <div class="p-3 border-t border-white/10">
              <button onclick="logoutToLanding()"
                class="w-full py-2.5 px-3 rounded-xl bg-white/10 hover:bg-rose-600 text-white text-xs font-semibold transition flex items-center justify-center gap-2 cursor-pointer">
                <i class="fa-solid fa-right-from-bracket"></i> Switch Portal / Logout
              </button>
            </div>
          </aside>

          <div class="flex-1 flex flex-col min-w-0">
            <header class="sticky top-0 z-20 bg-white border-b border-slate-200 px-4 sm:px-6 py-3 flex items-center justify-between gap-3 shadow-2xs">
              <div class="flex items-center gap-3">
                <button onclick="state.sidebarOpen=true; render();" class="lg:hidden p-2 rounded-xl bg-slate-100 text-slate-700 hover:bg-slate-200">
                  <i class="fa-solid fa-bars"></i>
                </button>
                <div>
                  <h1 class="text-sm sm:text-base font-bold text-slate-900">
                    ${navItems.find(n => n.id === state.studentTab)?.label || 'Student Dashboard'}
                  </h1>
                  <p class="text-[11px] text-slate-500 hidden sm:block">
                    Madhav Institute of Technology & Science, Gwalior • Session: July – Dec 2026
                  </p>
                </div>
              </div>

              <div class="flex items-center gap-2 sm:gap-3">
                ${unattemptedQuizzes > 0 ? `
                  <button onclick="setStudentTab('quiz')" class="hidden md:flex items-center gap-2 px-3 py-1.5 rounded-full bg-amber-100 hover:bg-amber-200 text-amber-900 border border-amber-300 text-xs font-bold transition cursor-pointer">
                    <span class="w-2 h-2 rounded-full bg-rose-600 animate-ping"></span>
                    ${unattemptedQuizzes} Live Quiz Active!
                  </button>
                ` : ''}
                <div class="px-3 py-1.5 rounded-xl bg-slate-100 text-slate-700 text-xs font-mono font-medium">
                  <i class="fa-regular fa-calendar text-indigo-600 mr-1.5"></i>${CURRENT_DATE_STR}
                </div>
              </div>
            </header>

            <div class="bg-mits-royal text-white text-xs py-1.5 px-4 overflow-hidden flex items-center">
              <span class="bg-amber-500 text-mits-navy font-extrabold text-[10px] px-2 py-0.5 rounded uppercase mr-3 shrink-0">Notice</span>
              <div class="overflow-hidden flex-1">
                <div class="animate-ticker space-x-10 text-slate-200">
                  ${state.announcements.map(a => `<span>${a}</span>`).join('')}
                </div>
              </div>
            </div>

            <main class="flex-1 p-4 sm:p-6 max-w-7xl w-full mx-auto space-y-6">
              ${renderStudentTabContent(aggAtt)}
            </main>
          </div>
        </div>
      `;
    }

    function renderStudentTabContent(aggAtt) {
      switch (state.studentTab) {
        case 'dashboard': return renderStudentDashboard(aggAtt);
        case 'attendance': return renderStudentAttendance(aggAtt);
        case 'syllabus': return renderStudentSyllabus();
        case 'quiz': return renderStudentQuiz();
        case 'exams': return renderStudentExams();
        case 'microproject': return renderStudentMicroProject();
        case 'services': return renderStudentServices();
        case 'clubs': return renderStudentClubs();
        case 'profile': return renderStudentProfileView();
        default: return renderStudentDashboard(aggAtt);
      }
    }

    function renderStudentDashboard(aggAtt) {
      const p = state.studentProfile;
      const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'];
      const rawDaySchedule = state.timetable[state.selectedDay] || [];
      const daySchedule = rawDaySchedule.filter(slot => !slot.batch || slot.batch === state.selectedBatch);
      const unattemptedQuizzes = state.quizzes.filter(q => q.status === 'not_started').length;

      return `
        <div class="grid grid-cols-1 xl:grid-cols-3 gap-6">
          <div class="xl:col-span-2 bg-white rounded-2xl border border-slate-200 shadow-xs overflow-hidden flex flex-col justify-between">
            <div class="bg-gradient-to-r from-mits-navy via-mits-royal to-indigo-900 px-6 py-4 text-white flex flex-wrap items-center justify-between gap-2">
              <div class="flex items-center gap-3">
                <div class="w-12 h-12 rounded-2xl bg-amber-400 text-mits-navy font-extrabold text-lg flex items-center justify-center shadow">
                  AG
                </div>
                <div>
                  <h2 class="text-lg font-bold">${p.name}</h2>
                  <p class="text-xs text-amber-300 font-mono">${p.enrollment} • ${p.email}</p>
                </div>
              </div>
              <button onclick="setStudentTab('profile')" class="px-3 py-1.5 rounded-full bg-white/15 hover:bg-white/25 text-white text-xs font-semibold transition cursor-pointer">
                <i class="fa-solid fa-user mr-1"></i> Full Profile
              </button>
            </div>
            <div class="p-6 grid grid-cols-1 sm:grid-cols-3 gap-4 text-xs">
              <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-100">
                <div class="text-slate-400 font-medium">Branch & Program</div>
                <div class="font-bold text-slate-800 mt-0.5">${p.branch}</div>
              </div>
              <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-100">
                <div class="text-slate-400 font-medium">Department</div>
                <div class="font-bold text-indigo-900 mt-0.5">${p.department}</div>
              </div>
              <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-100">
                <div class="text-slate-400 font-medium">Semester & Lab Group</div>
                <div class="font-bold text-slate-800 mt-0.5">${p.semester} • <span class="text-amber-700">${p.batch}</span></div>
              </div>
            </div>
          </div>

          <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs flex flex-col justify-between">
            <div>
              <div class="flex items-center justify-between mb-3">
                <h3 class="font-bold text-slate-900 text-sm flex items-center gap-2">
                  <i class="fa-solid fa-user-tie text-amber-600"></i> CIOT Department Leadership
                </h3>
                <span class="text-[11px] font-semibold px-2 py-0.5 rounded bg-amber-100 text-amber-800">EACE</span>
              </div>
              <div class="space-y-2.5">
                <div class="p-3 rounded-xl bg-indigo-50/70 border border-indigo-100">
                  <div class="text-[10px] font-bold text-indigo-700 uppercase">Dean, Centre for IoT</div>
                  <div class="font-bold text-slate-900 text-sm mt-0.5">Dr. Praveen Bansal</div>
                </div>
                <div class="p-3 rounded-xl bg-amber-50/70 border border-amber-100">
                  <div class="text-[10px] font-bold text-amber-800 uppercase">Branch Coordinators (EACE)</div>
                  <div class="font-bold text-slate-900 text-xs mt-0.5">Dr. Abhishek Sharma & Dr. Nand Kishore Joshi</div>
                </div>
              </div>
            </div>
            <div class="mt-3 pt-2.5 border-t border-slate-100 flex items-center justify-between text-[11px] text-slate-500">
              <span>Time Table w.e.f. 24.08.2026</span>
              <span class="font-mono font-semibold text-slate-700">Rooms M003–M005</span>
            </div>
          </div>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
          <div onclick="setStudentTab('attendance')" class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs hover:border-indigo-400 transition cursor-pointer">
            <div class="flex items-center justify-between">
              <span class="text-xs font-semibold text-slate-500">Overall Attendance</span>
              <span class="w-9 h-9 rounded-xl bg-emerald-100 text-emerald-700 flex items-center justify-center"><i class="fa-solid fa-chart-line"></i></span>
            </div>
            <div class="text-2xl font-extrabold text-slate-900 mt-2">${aggAtt}%</div>
            <div class="text-xs text-emerald-700 font-medium mt-1">
              <i class="fa-solid fa-shield-check mr-1"></i>Above 75% University Criteria
            </div>
          </div>

          <div onclick="setStudentTab('quiz')" class="bg-white rounded-2xl border border-amber-300 p-5 shadow-xs hover:shadow-md transition cursor-pointer">
            <div class="flex items-center justify-between">
              <span class="text-xs font-semibold text-amber-800">In-Class Live Quizzes</span>
              <span class="w-9 h-9 rounded-xl bg-amber-100 text-amber-700 flex items-center justify-center"><i class="fa-solid fa-stopwatch"></i></span>
            </div>
            <div class="text-2xl font-extrabold text-slate-900 mt-2">${unattemptedQuizzes > 0 ? `${unattemptedQuizzes} Active` : 'All Completed'}</div>
            <div class="text-xs text-amber-700 font-semibold mt-1">
              ${unattemptedQuizzes > 0 ? 'Tap to Launch Proctored Quiz →' : 'View Relative Grades →'}
            </div>
          </div>

          <div onclick="setStudentTab('exams')" class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs hover:border-indigo-400 transition cursor-pointer">
            <div class="flex items-center justify-between">
              <span class="text-xs font-semibold text-slate-500">4-Tier Exam Scorebook</span>
              <span class="w-9 h-9 rounded-xl bg-indigo-100 text-indigo-700 flex items-center justify-center"><i class="fa-solid fa-award"></i></span>
            </div>
            <div class="text-2xl font-extrabold text-slate-900 mt-2">87 / 100</div>
            <div class="text-xs text-indigo-700 font-medium mt-1">
              Central Proctored Quiz • Minor Pending
            </div>
          </div>
        </div>

        <div class="bg-white rounded-2xl border border-slate-200 shadow-xs overflow-hidden">
          <div class="p-5 border-b border-slate-200 flex flex-col lg:flex-row lg:items-center justify-between gap-4">
            <div>
              <div class="flex flex-wrap items-center gap-2">
                <span class="px-2.5 py-0.5 rounded-full bg-indigo-100 text-indigo-800 text-xs font-bold">
                  <i class="fa-solid fa-calendar-day mr-1"></i> Official Time Table (w.e.f. 24.08.2026)
                </span>
                <span class="text-xs text-slate-500 font-medium">B.Tech. EACE • I Year / I Sem</span>
              </div>
              <h3 class="text-base font-bold text-slate-900 mt-1">
                ${state.selectedDay} Class & Lab Schedule (${state.selectedBatch})
              </h3>
            </div>

            <div class="flex flex-wrap items-center gap-3">
              <div class="flex items-center bg-amber-50 border border-amber-200 p-1 rounded-xl" title="Theory lectures are common for all 80 EACE students; Labs split into Batch A (40) & Batch B (40)">
                <button onclick="state.selectedBatch='Batch A'; render();"
                  class="px-3 py-1 rounded-lg text-xs font-bold transition cursor-pointer ${state.selectedBatch === 'Batch A' ? 'bg-amber-500 text-mits-navy shadow-2xs' : 'text-amber-900 hover:bg-amber-100'}">
                  Lab Batch A
                </button>
                <button onclick="state.selectedBatch='Batch B'; render();"
                  class="px-3 py-1 rounded-lg text-xs font-bold transition cursor-pointer ${state.selectedBatch === 'Batch B' ? 'bg-amber-500 text-mits-navy shadow-2xs' : 'text-amber-900 hover:bg-amber-100'}">
                  Lab Batch B
                </button>
              </div>

              <div class="flex flex-wrap gap-1 bg-slate-100 p-1 rounded-xl">
                ${days.map(d => `
                  <button onclick="state.selectedDay='${d}'; render();"
                    class="px-3 py-1.5 rounded-lg text-xs font-semibold transition cursor-pointer ${state.selectedDay === d ? 'bg-mits-navy text-white shadow-xs' : 'text-slate-600 hover:text-slate-900'}">
                    ${d.slice(0, 3)}${d === 'Tuesday' ? ' •' : ''}
                  </button>
                `).join('')}
              </div>
            </div>
          </div>

          <div class="p-5 grid grid-cols-1 md:grid-cols-2 gap-4">
            ${daySchedule.map(slot => {
              if (slot.type === 'Break') {
                return `
                  <div class="p-3.5 rounded-2xl border border-dashed border-slate-300 bg-slate-50/70 flex items-center justify-between gap-4">
                    <div class="flex items-center gap-3">
                      <span class="text-xs font-mono font-bold text-slate-600 bg-slate-200 px-2.5 py-1 rounded-lg">${slot.time}</span>
                      <span class="text-xs font-bold text-slate-700"><i class="fa-solid fa-mug-hot text-amber-600 mr-1.5"></i>${slot.subject}</span>
                    </div>
                  </div>
                `;
              }
              const isOngoing = state.selectedDay === 'Tuesday' && slot.status === 'Ongoing';
              const badgeStyle = slot.type === 'Lab'
                ? 'bg-purple-100 text-purple-800'
                : slot.type === 'Project'
                ? 'bg-emerald-100 text-emerald-800'
                : slot.type === 'Presentation'
                ? 'bg-amber-100 text-amber-900'
                : 'bg-slate-200 text-slate-700';

              return `
                <div class="p-4 rounded-2xl border ${isOngoing ? 'border-amber-400 bg-amber-50/50 ring-2 ring-amber-400/20' : 'border-slate-200 bg-slate-50/50'} flex items-start justify-between gap-4">
                  <div class="space-y-1.5">
                    <div class="flex flex-wrap items-center gap-2">
                      <span class="text-xs font-mono font-bold text-indigo-900 bg-indigo-100 px-2.5 py-0.5 rounded-lg">${slot.time}</span>
                      <span class="text-xs font-bold px-2 py-0.5 rounded-lg ${badgeStyle}">${slot.type}</span>
                      ${isOngoing ? `<span class="text-[11px] font-bold px-2 py-0.5 rounded-full bg-amber-500 text-mits-navy animate-pulse">● ACTIVE SLOT</span>` : ''}
                    </div>
                    <div class="font-bold text-slate-900 text-sm">${slot.subject}</div>
                    <div class="text-xs text-slate-600 flex flex-wrap items-center gap-3">
                      <span><i class="fa-solid fa-user-tie text-slate-400 mr-1"></i>${slot.faculty}</span>
                      <span class="font-mono font-semibold text-amber-800"><i class="fa-solid fa-door-open text-amber-600 mr-1"></i>Room: ${slot.room}</span>
                    </div>
                  </div>
                  <div class="text-xs font-mono font-bold text-slate-500 shrink-0">${slot.code}</div>
                </div>
              `;
            }).join('')}
          </div>
        </div>
      `;
    }

    function renderStudentProfileView() {
      const p = state.studentProfile;
      return `
        <div class="max-w-4xl mx-auto bg-white rounded-2xl border border-slate-200 shadow-xs overflow-hidden">
          <div class="bg-gradient-to-r from-mits-navy via-mits-royal to-indigo-900 p-6 text-white flex flex-wrap items-center justify-between gap-4">
            <div class="flex items-center gap-4">
              <div class="w-16 h-16 rounded-2xl bg-amber-400 text-mits-navy font-extrabold text-2xl flex items-center justify-center shadow-lg">
                AG
              </div>
              <div>
                <h2 class="text-2xl font-extrabold">${p.name}</h2>
                <p class="text-xs text-amber-300 font-mono mt-0.5">${p.enrollment} • ${p.email}</p>
              </div>
            </div>
            <span class="px-3.5 py-1.5 rounded-full bg-emerald-500/20 border border-emerald-400/40 text-emerald-300 text-xs font-bold">
              <i class="fa-solid fa-circle-check mr-1.5"></i> Verified University Record
            </span>
          </div>

          <div class="p-6 space-y-6">
            <div>
              <h3 class="text-xs font-extrabold uppercase tracking-wider text-slate-400 mb-3">Academic Enrollment Details</h3>
              <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4 text-xs">
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Full Name</div>
                  <div class="font-bold text-slate-900 text-sm mt-0.5">${p.name}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Enrollment Number</div>
                  <div class="font-mono font-bold text-indigo-900 text-sm mt-0.5">${p.enrollment}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">University Email ID</div>
                  <div class="font-mono font-bold text-slate-800 mt-0.5">${p.email}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Branch</div>
                  <div class="font-bold text-slate-900 mt-0.5">${p.branch}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Department</div>
                  <div class="font-bold text-slate-900 mt-0.5">${p.department}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Semester & Lab Group</div>
                  <div class="font-bold text-amber-800 mt-0.5">${p.semester} • ${p.batch} (40 Students)</div>
                </div>
              </div>
            </div>

            <div class="pt-4 border-t border-slate-200">
              <h3 class="text-xs font-extrabold uppercase tracking-wider text-slate-400 mb-3">Personal & Residential Details</h3>
              <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4 text-xs">
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Father's Name</div>
                  <div class="font-bold text-slate-900 text-sm mt-0.5">${p.fatherName}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Hostel Residence</div>
                  <div class="font-bold text-slate-900 mt-0.5">${p.hostel}</div>
                </div>
                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-slate-400 font-medium">Registered Contact</div>
                  <div class="font-mono font-bold text-slate-900 mt-0.5">${p.phone}</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      `;
    }

    function adjustSimAttendance(code, dAtt, dTot) {
      const c = state.courses.find(x => x.code === code);
      if (!c) return;
      c.simDeltaAttended = (c.simDeltaAttended || 0) + dAtt;
      c.simDeltaTotal = (c.simDeltaTotal || 0) + dTot;
      render();
    }

    function resetAllSimulators() {
      state.courses.forEach(c => {
        c.simDeltaAttended = 0;
        c.simDeltaTotal = 0;
      });
      render();
      showToast("What-If Attendance Simulator reset to actual ERP records.", "info");
    }

    function renderStudentAttendance(aggAtt) {
      const dangerCourses = state.courses.filter(c => getAttendanceStats(c).pct < 75);

      return `
        <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs flex flex-col lg:flex-row items-start lg:items-center justify-between gap-6">
          <div>
            <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-indigo-50 text-indigo-800 text-xs font-bold mb-2">
              <i class="fa-solid fa-calculator"></i> MITS 75% Mandatory Attendance Criteria Engine
            </div>
            <h2 class="text-xl font-extrabold text-slate-900">Subject-Wise Attendance & Bunk / Recovery Predictor</h2>
            <p class="text-xs text-slate-500 mt-1">
              Synced live with Faculty ERP. Use the <strong>What-If Simulator</strong> on any subject card to test attending or missing upcoming classes.
            </p>
          </div>
          <div class="flex flex-wrap items-center gap-3 shrink-0">
            <div class="px-4 py-2.5 rounded-xl bg-slate-900 text-white text-center">
              <div class="text-[11px] text-slate-400">Overall Aggregate</div>
              <div class="text-xl font-extrabold text-amber-400">${aggAtt}%</div>
            </div>
            <div class="px-4 py-2.5 rounded-xl ${dangerCourses.length > 0 ? 'bg-rose-50 border border-rose-200 text-rose-900' : 'bg-emerald-50 border border-emerald-200 text-emerald-900'} text-center">
              <div class="text-[11px] font-semibold">Below 75% Subjects</div>
              <div class="text-xl font-extrabold">${dangerCourses.length} Course${dangerCourses.length === 1 ? '' : 's'}</div>
            </div>
            <button onclick="resetAllSimulators()" class="px-3.5 py-2.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-semibold transition cursor-pointer">
              <i class="fa-solid fa-rotate-left mr-1"></i> Reset Simulator
            </button>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
          ${state.courses.map(c => {
            const st = getAttendanceStats(c);
            const hasSim = (c.simDeltaTotal || 0) !== 0;
            return `
              <div class="bg-white rounded-2xl border ${st.isSafe ? 'border-slate-200' : 'border-rose-300 ring-2 ring-rose-500/10'} p-5 shadow-xs flex flex-col justify-between space-y-4">
                <div>
                  <div class="flex items-start justify-between gap-3">
                    <div>
                      <div class="flex items-center gap-2">
                        <span class="text-xs font-mono font-bold px-2 py-0.5 rounded bg-slate-100 text-slate-700">${c.code}</span>
                        <span class="text-[11px] font-semibold px-2 py-0.5 rounded ${c.type === 'Lab' ? 'bg-purple-100 text-purple-800' : 'bg-blue-50 text-blue-700'}">${c.type}</span>
                        ${hasSim ? `<span class="text-[10px] font-bold px-2 py-0.5 rounded bg-amber-100 text-amber-800">Simulated</span>` : ''}
                      </div>
                      <h3 class="font-bold text-slate-900 text-sm sm:text-base mt-1.5">${c.name}</h3>
                      <p class="text-xs text-slate-500 mt-0.5"><i class="fa-solid fa-chalkboard-user mr-1"></i>${c.faculty}</p>
                    </div>
                    <div class="text-right shrink-0">
                      <div class="text-2xl font-extrabold ${st.isSafe ? 'text-emerald-600' : 'text-rose-600'}">${st.pct.toFixed(1)}%</div>
                      <span class="inline-block text-[11px] font-bold px-2.5 py-0.5 rounded-full ${st.isSafe ? 'bg-emerald-100 text-emerald-800' : 'bg-rose-100 text-rose-800'}">
                        ${st.isSafe ? 'SAFE (≥75%)' : 'DANGER (<75%)'}
                      </span>
                    </div>
                  </div>

                  <div class="mt-4">
                    <div class="flex justify-between text-xs font-medium text-slate-600 mb-1">
                      <span>Attended: <strong>${st.att}</strong> / ${st.tot} classes</span>
                      <span class="text-slate-400">Target: 75.0%</span>
                    </div>
                    <div class="w-full h-3 bg-slate-100 rounded-full overflow-hidden relative">
                      <div class="h-full rounded-full transition-all duration-300 ${st.isSafe ? 'bg-emerald-500' : 'bg-rose-500'}" style="width: ${Math.min(100, st.pct)}%"></div>
                      <div class="absolute top-0 bottom-0 w-0.5 bg-slate-900" style="left: 75%" title="75% Threshold"></div>
                    </div>
                  </div>

                  <div class="mt-3.5 p-3 rounded-xl ${st.isSafe ? 'bg-emerald-50/80 border border-emerald-200 text-emerald-950' : 'bg-rose-50 border border-rose-200 text-rose-950'} text-xs flex items-center gap-2.5">
                    <i class="fa-solid ${st.isSafe ? 'fa-circle-check text-emerald-600' : 'fa-triangle-exclamation text-rose-600'} text-base"></i>
                    <div>${st.predictorText}</div>
                  </div>
                </div>

                <div class="pt-3 border-t border-slate-100 flex flex-wrap items-center justify-between gap-2 text-xs">
                  <span class="font-semibold text-slate-500">What-If Next Class:</span>
                  <div class="flex items-center gap-1.5">
                    <button onclick="adjustSimAttendance('${c.code}', 1, 1)" class="px-2.5 py-1.5 rounded-lg bg-emerald-100 hover:bg-emerald-200 text-emerald-900 font-semibold transition cursor-pointer">
                      + Attend 1
                    </button>
                    <button onclick="adjustSimAttendance('${c.code}', 0, 1)" class="px-2.5 py-1.5 rounded-lg bg-rose-100 hover:bg-rose-200 text-rose-900 font-semibold transition cursor-pointer">
                      - Bunk 1
                    </button>
                  </div>
                </div>
              </div>
            `;
          }).join('')}
        </div>
      `;
    }

    function previewStudyMaterial(title, courseName, unitNum, type, by, date) {
      openModal(`
        <div class="p-6">
          <div class="flex items-start justify-between gap-4 border-b border-slate-200 pb-4">
            <div>
              <span class="px-2.5 py-0.5 rounded-full bg-indigo-100 text-indigo-800 text-xs font-bold">${type} • Unit ${unitNum}</span>
              <h3 class="text-lg font-bold text-slate-900 mt-1">${title}</h3>
              <p class="text-xs text-slate-500 mt-0.5">${courseName} • Uploaded by ${by} (${date})</p>
            </div>
            <button onclick="closeModal()" class="text-slate-400 hover:text-slate-700 p-1"><i class="fa-solid fa-xmark text-lg"></i></button>
          </div>
          <div class="my-5 p-6 rounded-2xl bg-slate-50 border border-slate-200 text-center space-y-3">
            <div class="w-14 h-14 rounded-2xl bg-rose-100 text-rose-600 flex items-center justify-center text-2xl mx-auto">
              <i class="fa-solid fa-file-pdf"></i>
            </div>
            <div class="text-sm font-bold text-slate-800">MITS CIOT Digital Repository Document Ready</div>
            <p class="text-xs text-slate-500 max-w-md mx-auto">
              Official watermarked academic handout for 1st Semester EACE students (Enrollment: ${state.studentProfile.enrollment}).
            </p>
          </div>
          <div class="flex justify-end gap-2">
            <button onclick="closeModal()" class="px-4 py-2 rounded-xl border border-slate-300 text-xs font-semibold text-slate-700 hover:bg-slate-50">Close</button>
            <button onclick="closeModal(); showToast('Downloaded: ${title.replace(/'/g, '')}', 'success');" class="px-4 py-2 rounded-xl bg-mits-navy text-white text-xs font-semibold hover:bg-slate-800">
              <i class="fa-solid fa-download mr-1.5"></i> Download PDF
            </button>
          </div>
        </div>
      `);
    }

    function renderStudentSyllabus() {
      const selectedCourse = state.courses.find(c => c.code === state.selectedCourseCode) || state.courses[0];

      return `
        <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
          <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-5">
            <div>
              <h2 class="text-xl font-extrabold text-slate-900">NEP 5-Unit Curriculum & Faculty Study Repository</h2>
              <p class="text-xs text-slate-500 mt-0.5">Select a subject below to explore Unit 1–5 syllabus topics and faculty-uploaded PDFs & question banks.</p>
            </div>
            <span class="text-xs font-semibold px-3 py-1.5 rounded-xl bg-amber-50 border border-amber-200 text-amber-900">
              <i class="fa-solid fa-arrows-rotate mr-1"></i> Synced with Faculty ERP
            </span>
          </div>

          <div class="flex flex-wrap gap-2 pb-5 border-b border-slate-200">
            ${state.courses.map(c => `
              <button onclick="state.selectedCourseCode='${c.code}'; render();"
                class="px-3.5 py-2 rounded-xl text-xs font-semibold transition cursor-pointer border ${selectedCourse.code === c.code ? 'bg-mits-navy text-white border-mits-navy shadow-sm' : 'bg-slate-50 text-slate-700 border-slate-200 hover:bg-slate-100'}">
                <span class="font-mono opacity-75 mr-1">${c.code}</span> ${c.shortName}
              </button>
            `).join('')}
          </div>

          <div class="py-5 flex flex-wrap items-center justify-between gap-4">
            <div>
              <div class="flex items-center gap-2">
                <span class="px-2.5 py-0.5 rounded bg-indigo-100 text-indigo-800 font-mono text-xs font-bold">${selectedCourse.code}</span>
                <span class="px-2.5 py-0.5 rounded bg-slate-100 text-slate-700 text-xs font-semibold">${selectedCourse.credits} Credits • ${selectedCourse.type}</span>
              </div>
              <h3 class="text-lg font-bold text-slate-900 mt-1.5">${selectedCourse.name}</h3>
              <p class="text-xs text-slate-500">Course Faculty: <strong class="text-slate-700">${selectedCourse.faculty}</strong></p>
            </div>
            <div class="text-xs bg-slate-50 border border-slate-200 px-4 py-2.5 rounded-xl">
              Total Uploaded Resources: <strong class="text-indigo-700 text-sm ml-1">${selectedCourse.materials.length}</strong>
            </div>
          </div>

          <div class="space-y-4">
            ${selectedCourse.units.map(u => {
              const unitFiles = selectedCourse.materials.filter(m => m.unit === u.unit);
              return `
                <div class="rounded-2xl border border-slate-200 overflow-hidden bg-slate-50/40">
                  <div class="p-4 sm:p-5 bg-white flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                    <div>
                      <div class="flex items-center gap-2">
                        <span class="px-2.5 py-0.5 rounded-lg bg-amber-500 text-mits-navy text-xs font-extrabold">UNIT ${u.unit}</span>
                        <h4 class="font-bold text-slate-900 text-sm sm:text-base">${u.title}</h4>
                      </div>
                      <p class="text-xs text-slate-600 mt-1.5 leading-relaxed"><strong>Syllabus Topics:</strong> ${u.topics}</p>
                    </div>
                    <span class="text-xs font-semibold text-slate-500 shrink-0">${unitFiles.length} File${unitFiles.length === 1 ? '' : 's'}</span>
                  </div>

                  <div class="px-4 sm:px-5 py-3 bg-slate-50 border-t border-slate-100">
                    ${unitFiles.length > 0 ? `
                      <div class="grid grid-cols-1 md:grid-cols-2 gap-2.5">
                        ${unitFiles.map(f => `
                          <div class="p-3 rounded-xl bg-white border border-slate-200 flex items-center justify-between gap-3 hover:border-indigo-300 transition">
                            <div class="flex items-center gap-3 min-w-0">
                              <div class="w-9 h-9 rounded-lg ${f.type === 'Question Bank' ? 'bg-amber-100 text-amber-700' : 'bg-rose-100 text-rose-600'} flex items-center justify-center shrink-0">
                                <i class="fa-solid ${f.type === 'Question Bank' ? 'fa-clipboard-question' : 'fa-file-pdf'}"></i>
                              </div>
                              <div class="min-w-0">
                                <div class="text-xs font-bold text-slate-800 truncate">${f.title}</div>
                                <div class="text-[11px] text-slate-400">${f.type} • ${f.size} • ${f.date}</div>
                              </div>
                            </div>
                            <button onclick="previewStudyMaterial('${f.title.replace(/'/g, "\\'")}', '${selectedCourse.name.replace(/'/g, "\\'")}', ${u.unit}, '${f.type}', '${f.by}', '${f.date}')"
                              class="px-3 py-1.5 rounded-lg bg-indigo-50 hover:bg-indigo-600 text-indigo-700 hover:text-white text-xs font-semibold transition shrink-0 cursor-pointer">
                              Open
                            </button>
                          </div>
                        `).join('')}
                      </div>
                    ` : `
                      <p class="text-xs text-slate-400 italic py-1">No lecture PDFs uploaded for Unit ${u.unit} yet.</p>
                    `}
                  </div>
                </div>
              `;
            }).join('')}
          </div>
        </div>
      `;
    }

    function getActiveQuiz() {
      return state.quizzes[state.activeQuizIndex] || state.quizzes[0];
    }

    function startLiveQuiz() {
      const qz = getActiveQuiz();
      if (!qz) return;
      try {
        if (document.documentElement.requestFullscreen) {
          document.documentElement.requestFullscreen().catch(() => {});
        }
      } catch (e) {}

      qz.status = 'in_progress';
      qz.strikes = 0;
      qz.timeLeft = (qz.durationMins || 10) * 60;
      render();

      if (quizInterval) clearInterval(quizInterval);
      quizInterval = setInterval(() => {
        const cur = getActiveQuiz();
        if (!cur || cur.status !== 'in_progress') {
          clearInterval(quizInterval);
          return;
        }
        cur.timeLeft--;
        const timerEl = document.getElementById('quiz-timer-display');
        if (timerEl) {
          const mins = String(Math.floor(cur.timeLeft / 60)).padStart(2, '0');
          const secs = String(cur.timeLeft % 60).padStart(2, '0');
          timerEl.textContent = `${mins}:${secs}`;
        }
        if (cur.timeLeft <= 0) {
          submitLiveQuiz(true);
        }
      }, 1000);
    }

    function triggerAntiCheatStrike(reason = "Tab Switch / Window Focus Lost") {
      const qz = getActiveQuiz();
      if (!qz || qz.status !== 'in_progress') return;
      qz.strikes += 1;

      if (qz.strikes === 1) {
        openModal(`
          <div class="p-6 text-center space-y-4">
            <div class="w-16 h-16 rounded-full bg-amber-100 text-amber-600 flex items-center justify-center text-3xl mx-auto animate-bounce">
              <i class="fa-solid fa-triangle-exclamation"></i>
            </div>
            <span class="px-3 py-1 rounded-full bg-amber-100 text-amber-900 text-xs font-extrabold uppercase">Anti-Cheating Strike 1 of 2</span>
            <h3 class="text-xl font-extrabold text-slate-900">Proctoring Violation Detected!</h3>
            <p class="text-sm text-slate-600">
              Reason: <strong>${reason}</strong>.<br/>
              Under MITS CIOT strict anti-cheating policy, <strong>one more violation (Strike 2)</strong> will immediately terminate and lock your quiz attempt with zero recovery.
            </p>
            <button onclick="closeModal()" class="w-full py-3 rounded-xl bg-mits-navy text-white font-bold text-sm hover:bg-slate-800 cursor-pointer">
              I Understand — Return to Quiz
            </button>
          </div>
        `);
        render();
      } else if (qz.strikes >= 2) {
        if (quizInterval) clearInterval(quizInterval);
        qz.status = 'terminated';
        if (document.fullscreenElement) {
          document.exitFullscreen().catch(() => {});
        }
        openModal(`
          <div class="p-6 text-center space-y-4">
            <div class="w-16 h-16 rounded-full bg-rose-100 text-rose-600 flex items-center justify-center text-3xl mx-auto">
              <i class="fa-solid fa-ban"></i>
            </div>
            <span class="px-3 py-1 rounded-full bg-rose-100 text-rose-800 text-xs font-extrabold uppercase">Strike 2 of 2 • Attempt Blocked</span>
            <h3 class="text-xl font-extrabold text-rose-900">Quiz Attempt Auto-Terminated</h3>
            <p class="text-sm text-slate-600">
              Multiple tab/window switches were recorded for <strong>${state.studentProfile.enrollment}</strong>. Your session has been locked and logged for <strong>${qz.faculty}</strong>.
            </p>
            <button onclick="closeModal()" class="w-full py-3 rounded-xl bg-rose-700 text-white font-bold text-sm hover:bg-rose-800 cursor-pointer">
              Close Notice
            </button>
          </div>
        `);
        render();
      }
    }

    document.addEventListener('visibilitychange', () => {
      const qz = getActiveQuiz();
      if (document.hidden && state.currentPortal === 'student' && state.studentTab === 'quiz' && qz && qz.status === 'in_progress') {
        triggerAntiCheatStrike("Browser Tab Switched / Minimized");
      }
    });

    function toggleQuizMultiAnswer(qKey, idx) {
      const qz = getActiveQuiz();
      if (!qz.answers[qKey]) qz.answers[qKey] = [];
      const arr = qz.answers[qKey];
      const pos = arr.indexOf(idx);
      if (pos === -1) arr.push(idx);
      else arr.splice(pos, 1);
      render();
    }

    function submitLiveQuiz(autoTimeout = false) {
      const qz = getActiveQuiz();
      if (!qz) return;
      if (quizInterval) clearInterval(quizInterval);
      if (document.fullscreenElement) {
        document.exitFullscreen().catch(() => {});
      }

      let score = 0;
      qz.questions.forEach((q, idx) => {
        const key = `q${idx + 1}`;
        const ans = qz.answers[key];
        if (q.type === 'mcq') {
          if (ans === (q.correctIndex ?? 0)) score += (q.points || 2);
        } else if (q.type === 'checkbox') {
          const valid = q.correctIndices || [0];
          const chosen = Array.isArray(ans) ? ans : [];
          const hasWrong = chosen.some(x => !valid.includes(x));
          if (!hasWrong && chosen.length === valid.length) score += (q.points || 4);
          else if (!hasWrong && chosen.length > 0) score += Math.round((q.points || 4) / 2);
        } else if (q.type === 'integer') {
          if (parseInt(ans, 10) === parseInt(q.correct, 10)) score += (q.points || 3);
        } else if (q.type === 'match') {
          let rightCount = 0;
          const userMap = ans || {};
          (q.left || []).forEach(pair => {
            if (userMap[pair.key] === pair.match) rightCount++;
          });
          if (rightCount === (q.left || []).length) score += (q.points || 4);
          else if (rightCount > 0) score += Math.round(((q.points || 4) * rightCount) / (q.left || []).length);
        }
      });

      qz.score = score;
      qz.status = 'submitted';

      const ratio = score / Math.max(1, qz.totalMarks);
      if (ratio >= 0.88) { qz.grade = "A+"; qz.percentile = "96.2%ile (Top 4 of 80)"; }
      else if (ratio >= 0.75) { qz.grade = "A"; qz.percentile = "84.5%ile (Top 13 of 80)"; }
      else if (ratio >= 0.60) { qz.grade = "B+"; qz.percentile = "68.0%ile (Rank 26 of 80)"; }
      else if (ratio >= 0.45) { qz.grade = "B"; qz.percentile = "48.8%ile (Rank 41 of 80)"; }
      else { qz.grade = "C"; qz.percentile = "24.0%ile (Rank 61 of 80)"; }

      const course = state.courses.find(c => c.code === qz.courseCode);
      if (course) {
        course.classQuizMarks = score;
        course.classQuizMax = qz.totalMarks;
      }

      render();
      showToast(autoTimeout ? "Time expired! Quiz auto-submitted." : "Quiz submitted! Relative grade computed & synced to Scorebook.", "success");
      setTimeout(renderGradingBellCurve, 80);
    }

    function resetDemoQuiz() {
      const qz = getActiveQuiz();
      if (!qz) return;
      qz.status = 'not_started';
      qz.strikes = 0;
      qz.timeLeft = (qz.durationMins || 10) * 60;
      qz.answers = {};
      render();
    }

    function renderGradingBellCurve() {
      const canvas = document.getElementById('relativeGradingChart');
      if (!canvas) return;
      if (gradingChartInstance) gradingChartInstance.destroy();
      gradingChartInstance = new Chart(canvas, {
        type: 'bar',
        data: {
          labels: ['F (<35%)', 'C (35–49%)', 'B (50–64%)', 'B+ (65–74%)', 'A (75–87%)', 'A+ (≥88%)'],
          datasets: [{
            label: 'Students in EACE Class (N = 80)',
            data: [3, 11, 22, 24, 14, 6],
            backgroundColor: ['#f43f5e', '#94a3b8', '#60a5fa', '#3b82f6', '#1e3a8a', '#f59e0b'],
            borderRadius: 8
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { display: false } },
          scales: {
            y: { beginAtZero: true, title: { display: true, text: 'No. of Students (Total 80)' } }
          }
        }
      });
    }

    function renderStudentQuiz() {
      const qz = getActiveQuiz();
      if (!qz) return "";

      const quizSelectorHeader = state.quizzes.length > 1 ? `
        <div class="bg-white rounded-2xl border border-slate-200 p-4 shadow-2xs flex flex-wrap items-center justify-between gap-3 mb-5">
          <span class="text-xs font-bold text-slate-700"><i class="fa-solid fa-layer-group text-indigo-600 mr-1.5"></i>Available Faculty Quizzes (${state.quizzes.length}):</span>
          <div class="flex flex-wrap gap-2">
            ${state.quizzes.map((item, idx) => `
              <button onclick="state.activeQuizIndex=${idx}; render(); if(state.quizzes[${idx}].status==='submitted') setTimeout(renderGradingBellCurve,50);"
                class="px-3 py-1.5 rounded-xl text-xs font-bold cursor-pointer border ${state.activeQuizIndex === idx ? 'bg-mits-navy text-white border-mits-navy' : 'bg-slate-50 text-slate-700 border-slate-200 hover:bg-slate-100'}">
                ${item.courseCode}: ${item.title.slice(0, 28)}...
                <span class="ml-1 px-1.5 py-0.2 rounded text-[10px] ${item.status === 'submitted' ? 'bg-emerald-500 text-white' : 'bg-amber-400 text-mits-navy'}">${item.status === 'submitted' ? 'Done' : 'LIVE'}</span>
              </button>
            `).join('')}
          </div>
        </div>
      ` : '';

      if (qz.status === 'not_started') {
        return `
          ${quizSelectorHeader}
          <div class="max-w-3xl mx-auto bg-white rounded-2xl border border-slate-200 shadow-md overflow-hidden">
            <div class="bg-gradient-to-r from-mits-navy to-indigo-900 p-6 text-white">
              <div class="flex items-center justify-between gap-2">
                <span class="px-3 py-1 rounded-full bg-amber-400 text-mits-navy text-xs font-extrabold uppercase tracking-wider">
                  ● Live Classroom Assessment
                </span>
                <span class="font-mono text-xs text-amber-300">Room M003 • Class Strength: 80</span>
              </div>
              <h2 class="text-xl sm:text-2xl font-extrabold mt-3">${qz.title}</h2>
              <p class="text-xs text-slate-300 mt-1">${qz.courseCode} – ${qz.courseName} • Faculty: <strong>${qz.faculty}</strong></p>
            </div>

            <div class="p-6 space-y-5">
              <div class="grid grid-cols-3 gap-3 text-center">
                <div class="p-3 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-xs text-slate-500">Duration</div>
                  <div class="text-base font-extrabold text-slate-900 mt-0.5">${qz.durationMins || 10} Mins</div>
                </div>
                <div class="p-3 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-xs text-slate-500">Questions</div>
                  <div class="text-base font-extrabold text-slate-900 mt-0.5">${qz.questions.length} Items</div>
                </div>
                <div class="p-3 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-xs text-slate-500">Max Marks</div>
                  <div class="text-base font-extrabold text-indigo-900 mt-0.5">${qz.totalMarks} Pts</div>
                </div>
              </div>

              <div class="p-5 rounded-2xl bg-rose-50 border border-rose-200 space-y-2.5">
                <h3 class="text-sm font-extrabold text-rose-900 flex items-center gap-2">
                  <i class="fa-solid fa-shield-halved text-rose-600"></i> Strict Proctoring & 2-Strike Anti-Cheating Enforcement
                </h3>
                <ul class="text-xs text-rose-900/90 space-y-1.5 list-disc pl-5">
                  <li><strong>DND & Fullscreen Lock:</strong> Quiz launches in Fullscreen Proctored Mode. Do Not Disturb (DND) must remain active.</li>
                  <li><strong>2-Strike Tab-Switch Rule:</strong> Switching browser tabs, minimizing, or opening split-screen triggers <strong>Strike 1</strong>. A second switch (<strong>Strike 2</strong>) immediately terminates and blocks your submission.</li>
                  <li><strong>Relative Grading Engine:</strong> Grades (A+, A, B+, B, C) are normalized against the highest score across all 80 EACE students.</li>
                </ul>
              </div>

              <button onclick="startLiveQuiz()"
                class="w-full py-3.5 px-6 rounded-xl bg-mits-navy hover:bg-indigo-900 text-white font-bold text-sm shadow-lg transition flex items-center justify-center gap-2 cursor-pointer">
                <i class="fa-solid fa-expand"></i> Enable Fullscreen DND & Start Proctored Quiz
              </button>
            </div>
          </div>
        `;
      }

      if (qz.status === 'terminated') {
        return `
          ${quizSelectorHeader}
          <div class="max-w-xl mx-auto bg-white rounded-2xl border-2 border-rose-400 p-8 text-center shadow-lg space-y-4">
            <div class="w-16 h-16 rounded-full bg-rose-100 text-rose-600 flex items-center justify-center text-3xl mx-auto">
              <i class="fa-solid fa-lock"></i>
            </div>
            <h2 class="text-xl font-extrabold text-rose-900">Quiz Session Blocked (2/2 Strikes Recorded)</h2>
            <p class="text-xs text-slate-600 leading-relaxed">
              Your attempt for <strong>${qz.title}</strong> was automatically terminated due to repeated window/tab switching violations. Incident reported to <strong>${qz.faculty}</strong>.
            </p>
            <button onclick="resetDemoQuiz()" class="px-4 py-2 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-semibold cursor-pointer">
              <i class="fa-solid fa-rotate-left mr-1"></i> Reset Quiz Demo
            </button>
          </div>
        `;
      }

      if (qz.status === 'submitted') {
        return `
          ${quizSelectorHeader}
          <div class="space-y-6">
            <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
              <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-5 border-b border-slate-200">
                <div>
                  <span class="px-3 py-1 rounded-full bg-emerald-100 text-emerald-800 text-xs font-bold">
                    <i class="fa-solid fa-circle-check mr-1"></i> Synchronized Class Result Released
                  </span>
                  <h2 class="text-xl font-extrabold text-slate-900 mt-2">${qz.title}</h2>
                  <p class="text-xs text-slate-500">Relative Grading Cohort: EACE 1st Sem (N = 80 Students)</p>
                </div>
                <button onclick="resetDemoQuiz()" class="px-3.5 py-2 rounded-xl bg-slate-100 hover:bg-slate-200 text-xs font-semibold text-slate-700 self-start cursor-pointer">
                  <i class="fa-solid fa-rotate-left mr-1"></i> Retake Demo Quiz
                </button>
              </div>

              <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 my-5">
                <div class="p-4 rounded-2xl bg-indigo-50 border border-indigo-200">
                  <div class="text-xs font-semibold text-indigo-700">Your Raw Score</div>
                  <div class="text-2xl font-extrabold text-indigo-950 mt-1">${qz.score} <span class="text-sm font-normal">/ ${qz.totalMarks}</span></div>
                </div>
                <div class="p-4 rounded-2xl bg-amber-50 border border-amber-300">
                  <div class="text-xs font-semibold text-amber-800">Relative Letter Grade</div>
                  <div class="text-2xl font-extrabold text-amber-900 mt-1">${qz.grade}</div>
                </div>
                <div class="p-4 rounded-2xl bg-slate-50 border border-slate-200">
                  <div class="text-xs font-semibold text-slate-500">Class Highest / Mean</div>
                  <div class="text-xl font-extrabold text-slate-900 mt-1">${qz.totalMarks}.0 <span class="text-xs font-normal text-slate-500">/ ${(qz.totalMarks * 0.66).toFixed(1)} Avg</span></div>
                </div>
                <div class="p-4 rounded-2xl bg-emerald-50 border border-emerald-200">
                  <div class="text-xs font-semibold text-emerald-700">Class Standing</div>
                  <div class="text-sm font-extrabold text-emerald-950 mt-2">${qz.percentile}</div>
                </div>
              </div>

              <div class="p-4 rounded-2xl border border-slate-200 bg-slate-50/50">
                <h4 class="text-xs font-bold text-slate-700 uppercase mb-3">Relative Grade Distribution Across 80 Students</h4>
                <div class="h-56">
                  <canvas id="relativeGradingChart"></canvas>
                </div>
              </div>
            </div>
          </div>
        `;
      }

      const mins = String(Math.floor(qz.timeLeft / 60)).padStart(2, '0');
      const secs = String(qz.timeLeft % 60).padStart(2, '0');

      return `
        <div class="max-w-4xl mx-auto space-y-5">
          <div class="bg-mits-navy text-white rounded-2xl p-4 shadow-lg flex flex-wrap items-center justify-between gap-3">
            <div class="flex items-center gap-3">
              <span class="px-2.5 py-1 rounded-lg bg-emerald-500/20 border border-emerald-400/40 text-emerald-300 text-xs font-bold">
                <i class="fa-solid fa-shield-halved mr-1"></i> DND Proctoring Active
              </span>
              <span class="text-xs font-bold px-2.5 py-1 rounded-lg ${qz.strikes > 0 ? 'bg-rose-600 text-white' : 'bg-white/10 text-slate-200'}">
                Strikes: ${qz.strikes} / 2
              </span>
            </div>

            <div class="flex items-center gap-3">
              <button onclick="triggerAntiCheatStrike('Simulated Tab Switch Button')"
                class="px-3 py-1.5 rounded-xl bg-rose-600/90 hover:bg-rose-600 text-white text-xs font-semibold transition cursor-pointer">
                <i class="fa-solid fa-bug mr-1"></i> Simulate Tab Switch (Test 2-Strike)
              </button>
              <div class="px-3.5 py-1.5 rounded-xl bg-amber-400 text-mits-navy font-mono font-extrabold text-sm">
                <i class="fa-regular fa-clock mr-1"></i><span id="quiz-timer-display">${mins}:${secs}</span>
              </div>
            </div>
          </div>

          ${qz.questions.map((q, idx) => {
            const qKey = `q${idx + 1}`;
            if (q.type === 'mcq') {
              return `
                <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3">
                  <div class="flex items-center justify-between">
                    <span class="text-xs font-bold text-indigo-700 uppercase">Question ${idx + 1} • Single Choice MCQ</span>
                    <span class="px-2.5 py-0.5 rounded-full bg-slate-100 text-slate-700 text-xs font-bold">${q.points} Points</span>
                  </div>
                  <p class="text-sm font-semibold text-slate-900">${q.text}</p>
                  <div class="space-y-2 pt-1">
                    ${q.options.map((opt, i) => `
                      <label class="flex items-center gap-3 p-3 rounded-xl border ${qz.answers[qKey] === i ? 'border-indigo-600 bg-indigo-50' : 'border-slate-200 hover:bg-slate-50'} cursor-pointer text-xs sm:text-sm">
                        <input type="radio" name="${qKey}" ${qz.answers[qKey] === i ? 'checked' : ''} onchange="getActiveQuiz().answers['${qKey}']=${i}; render();" class="accent-indigo-700" />
                        <span>${opt}</span>
                      </label>
                    `).join('')}
                  </div>
                </div>
              `;
            } else if (q.type === 'checkbox') {
              const selectedArr = qz.answers[qKey] || [];
              return `
                <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3">
                  <div class="flex items-center justify-between">
                    <span class="text-xs font-bold text-purple-700 uppercase">Question ${idx + 1} • Multiple-Correct Checkbox</span>
                    <span class="px-2.5 py-0.5 rounded-full bg-slate-100 text-slate-700 text-xs font-bold">${q.points} Points</span>
                  </div>
                  <p class="text-sm font-semibold text-slate-900">${q.text}</p>
                  <div class="space-y-2 pt-1">
                    ${q.options.map((opt, i) => {
                      const checked = selectedArr.includes(i);
                      return `
                        <label class="flex items-center gap-3 p-3 rounded-xl border ${checked ? 'border-purple-600 bg-purple-50' : 'border-slate-200 hover:bg-slate-50'} cursor-pointer text-xs sm:text-sm">
                          <input type="checkbox" ${checked ? 'checked' : ''} onchange="toggleQuizMultiAnswer('${qKey}', ${i})" class="accent-purple-700 w-4 h-4" />
                          <span>${opt}</span>
                        </label>
                      `;
                    }).join('')}
                  </div>
                </div>
              `;
            } else if (q.type === 'integer') {
              return `
                <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3">
                  <div class="flex items-center justify-between">
                    <span class="text-xs font-bold text-amber-700 uppercase">Question ${idx + 1} • Integer / Numerical Value</span>
                    <span class="px-2.5 py-0.5 rounded-full bg-slate-100 text-slate-700 text-xs font-bold">${q.points} Points</span>
                  </div>
                  <p class="text-sm font-semibold text-slate-900">${q.text}</p>
                  <div class="max-w-xs pt-1">
                    <input type="number" value="${qz.answers[qKey] || ''}" oninput="getActiveQuiz().answers['${qKey}']=this.value" placeholder="Enter integer answer"
                      class="w-full px-4 py-2.5 rounded-xl border border-slate-300 font-mono text-sm focus:ring-2 focus:ring-amber-500" />
                  </div>
                </div>
              `;
            } else if (q.type === 'match') {
              const userMap = qz.answers[qKey] || {};
              return `
                <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3">
                  <div class="flex items-center justify-between">
                    <span class="text-xs font-bold text-teal-700 uppercase">Question ${idx + 1} • Match-the-Column</span>
                    <span class="px-2.5 py-0.5 rounded-full bg-slate-100 text-slate-700 text-xs font-bold">${q.points} Points</span>
                  </div>
                  <p class="text-sm font-semibold text-slate-900">${q.text}</p>
                  <div class="space-y-3 pt-2">
                    ${q.left.map(item => `
                      <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 items-center p-3 rounded-xl bg-slate-50 border border-slate-200">
                        <div class="text-xs font-bold text-slate-800"><span class="px-2 py-0.5 rounded bg-mits-navy text-white mr-2">${item.key}</span>${item.label}</div>
                        <select onchange="if(!getActiveQuiz().answers['${qKey}']) getActiveQuiz().answers['${qKey}']={}; getActiveQuiz().answers['${qKey}']['${item.key}']=this.value"
                          class="w-full px-3 py-2 rounded-xl border border-slate-300 bg-white text-xs font-medium">
                          <option value="">-- Select Matching Principle --</option>
                          ${q.rightOptions.map(ro => `
                            <option value="${ro}" ${userMap[item.key] === ro ? 'selected' : ''}>${ro}</option>
                          `).join('')}
                        </select>
                      </div>
                    `).join('')}
                  </div>
                </div>
              `;
            }
            return '';
          }).join('')}

          <div class="flex justify-end">
            <button onclick="submitLiveQuiz(false)"
              class="px-8 py-3.5 rounded-xl bg-emerald-700 hover:bg-emerald-600 text-white font-bold text-sm shadow-lg transition cursor-pointer">
              <i class="fa-solid fa-paper-plane mr-2"></i> Submit Quiz & Compute Relative Grade
            </button>
          </div>
        </div>
      `;
    }

    function renderStudentExams() {
      const theoryCourses = state.courses.filter(c => c.type === 'Theory');
      const labCourses = state.courses.filter(c => c.type === 'Lab');
      const centralTotal = theoryCourses.reduce((acc, c) => acc + (c.centralQuiz || 0), 0);
      const labMinorTotal = labCourses.reduce((acc, c) => acc + (c.minorMarks || 0), 0);

      return `
        <div class="space-y-6">
          <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
            <div class="flex flex-wrap items-center justify-between gap-2">
              <div>
                <h2 class="text-xl font-extrabold text-slate-900">4-Tier Examination & Continuous Evaluation Scorebook</h2>
                <p class="text-xs text-slate-500 mt-1">Synced directly with Faculty Marks Entry across (1) Class Quizzes, (2) Central Proctored Quiz, (3) Minor Exams & (4) Major Exams.</p>
              </div>
              <span class="px-3 py-1 rounded-full bg-indigo-50 text-indigo-800 text-xs font-bold">Session: July – Dec 2026</span>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3.5 mt-5">
              <div class="p-4 rounded-2xl bg-indigo-50 border border-indigo-200">
                <div class="text-[11px] font-extrabold text-indigo-700 uppercase">Tier 1 • Continuous</div>
                <div class="font-bold text-slate-900 text-sm mt-1">In-Class Quizzes</div>
                <div class="text-xs text-indigo-900 font-semibold mt-1">✓ Live Synced with Faculty</div>
              </div>
              <div class="p-4 rounded-2xl bg-emerald-50 border border-emerald-200">
                <div class="text-[11px] font-extrabold text-emerald-700 uppercase">Tier 2 • Sept 20, 2026</div>
                <div class="font-bold text-slate-900 text-sm mt-1">Central Proctored Quiz</div>
                <div class="text-xs text-emerald-800 font-semibold mt-1">✓ Total: ${centralTotal} / 100</div>
              </div>
              <div class="p-4 rounded-2xl bg-amber-50 border border-amber-300">
                <div class="text-[11px] font-extrabold text-amber-800 uppercase">Tier 3 • Oct & Nov 2–6</div>
                <div class="font-bold text-slate-900 text-sm mt-1">Lab & Theory Minors</div>
                <div class="text-xs text-amber-900 font-semibold mt-1">Labs: ${labMinorTotal}/90 • Theory: Pending</div>
              </div>
              <div class="p-4 rounded-2xl bg-slate-100 border border-slate-200">
                <div class="text-[11px] font-extrabold text-slate-600 uppercase">Tier 4 • Dec 4 & Dec 10</div>
                <div class="font-bold text-slate-900 text-sm mt-1">Lab & Theory Majors</div>
                <div class="text-xs text-slate-700 font-semibold mt-1">📅 CBT (1.5 Hrs / Paper)</div>
              </div>
            </div>
          </div>

          <div class="bg-white rounded-2xl border border-slate-200 shadow-xs overflow-hidden">
            <div class="p-5 border-b border-slate-200 flex items-center justify-between">
              <h3 class="text-base font-extrabold text-slate-900">Subject-Wise 4-Tier Consolidated Marksheet (${state.studentProfile.enrollment})</h3>
              <span class="text-xs text-slate-500">Auto-updates when faculty saves marks</span>
            </div>
            <div class="overflow-x-auto">
              <table class="w-full text-left text-xs">
                <thead class="bg-mits-navy text-white uppercase">
                  <tr>
                    <th class="py-3 px-4">Code & Course</th>
                    <th class="py-3 px-4">1. Class Quiz</th>
                    <th class="py-3 px-4">2. Central Quiz (20)</th>
                    <th class="py-3 px-4">3. Minor Exam (30)</th>
                    <th class="py-3 px-4">4. Major Exam (50)</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-slate-100">
                  ${state.courses.map(c => `
                    <tr class="hover:bg-slate-50/70">
                      <td class="py-3.5 px-4">
                        <div class="font-mono text-[11px] font-bold text-indigo-800">${c.code} <span class="text-slate-400">(${c.type})</span></div>
                        <div class="font-bold text-slate-900">${c.name}</div>
                      </td>
                      <td class="py-3.5 px-4 font-mono font-bold text-indigo-900">
                        ${c.classQuizMarks} / ${c.classQuizMax}
                      </td>
                      <td class="py-3.5 px-4 font-mono font-bold text-emerald-700">
                        ${c.centralQuiz !== null ? `${c.centralQuiz} / 20` : '<span class="text-slate-400 font-sans font-normal">N/A (Lab)</span>'}
                      </td>
                      <td class="py-3.5 px-4">
                        ${c.minorMarks !== null
                          ? `<span class="font-mono font-bold text-emerald-700 bg-emerald-50 px-2.5 py-1 rounded-lg">${c.minorMarks} / 30</span>`
                          : `<span class="px-2.5 py-1 rounded-lg bg-amber-100 text-amber-900 font-bold text-[11px]">${c.minorStatus}</span>`}
                      </td>
                      <td class="py-3.5 px-4">
                        ${c.majorMarks !== null
                          ? `<span class="font-mono font-bold text-indigo-800 bg-indigo-50 px-2.5 py-1 rounded-lg">${c.majorMarks} / 50</span>`
                          : `<span class="text-slate-500 font-medium">${c.majorStatus}</span>`}
                      </td>
                    </tr>
                  `).join('')}
                </tbody>
              </table>
            </div>
          </div>

          <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
              <div class="flex items-center justify-between mb-4">
                <div>
                  <span class="text-xs font-bold text-indigo-700 uppercase">Starts Dec 4, 2026</span>
                  <h3 class="text-base font-bold text-slate-900">Lab Major Practical Schedule (${state.selectedBatch})</h3>
                </div>
                <i class="fa-solid fa-flask text-indigo-600 text-xl"></i>
              </div>
              <div class="overflow-x-auto">
                <table class="w-full text-left text-xs">
                  <thead class="bg-slate-50 text-slate-500 uppercase border-b border-slate-200">
                    <tr>
                      <th class="py-2.5 px-3">Date & Shift</th>
                      <th class="py-2.5 px-3">Lab Course</th>
                      <th class="py-2.5 px-3">Room</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-slate-100">
                    <tr>
                      <td class="py-3 px-3 font-mono font-semibold">Dec 04, 2026 (10 AM)</td>
                      <td class="py-3 px-3 font-bold text-slate-800">33261106(A) – Computer Programming Lab</td>
                      <td class="py-3 px-3 font-mono text-slate-600">M005</td>
                    </tr>
                    <tr>
                      <td class="py-3 px-3 font-mono font-semibold">Dec 05, 2026 (10 AM)</td>
                      <td class="py-3 px-3 font-bold text-slate-800">33261107(A) – Embedded Programming Lab</td>
                      <td class="py-3 px-3 font-mono text-slate-600">M004</td>
                    </tr>
                    <tr>
                      <td class="py-3 px-3 font-mono font-semibold">Dec 07, 2026 (02 PM)</td>
                      <td class="py-3 px-3 font-bold text-slate-800">33261110 – Language Lab</td>
                      <td class="py-3 px-3 font-mono text-slate-600">M003</td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>

            <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
              <div class="flex items-center justify-between mb-4">
                <div>
                  <span class="text-xs font-bold text-amber-700 uppercase">Starts Dec 10, 2026</span>
                  <h3 class="text-base font-bold text-slate-900">Theory Major Exams (Computer-Based • 1.5 Hrs)</h3>
                </div>
                <i class="fa-solid fa-desktop text-amber-600 text-xl"></i>
              </div>
              <div class="overflow-x-auto">
                <table class="w-full text-left text-xs">
                  <thead class="bg-slate-50 text-slate-500 uppercase border-b border-slate-200">
                    <tr>
                      <th class="py-2.5 px-3">Date & Window</th>
                      <th class="py-2.5 px-3">Code & Subject</th>
                      <th class="py-2.5 px-3">CBT Center</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-slate-100">
                    <tr><td class="py-2.5 px-3 font-mono">Dec 10 (10:00–11:30)</td><td class="py-2.5 px-3 font-bold">33261101 – Intro to ECE</td><td class="py-2.5 px-3">CBT Hall-A</td></tr>
                    <tr><td class="py-2.5 px-3 font-mono">Dec 12 (10:00–11:30)</td><td class="py-2.5 px-3 font-bold">33261102 – Computer Programming</td><td class="py-2.5 px-3">CBT Hall-A</td></tr>
                    <tr><td class="py-2.5 px-3 font-mono">Dec 14 (10:00–11:30)</td><td class="py-2.5 px-3 font-bold">33261103 – Basics of Embedded Programming</td><td class="py-2.5 px-3">CBT Hall-B</td></tr>
                    <tr><td class="py-2.5 px-3 font-mono">Dec 16 (10:00–11:30)</td><td class="py-2.5 px-3 font-bold">33261104 – Digital Logic Design</td><td class="py-2.5 px-3">CBT Hall-A</td></tr>
                    <tr><td class="py-2.5 px-3 font-mono">Dec 18 (10:00–11:30)</td><td class="py-2.5 px-3 font-bold">33261105 – Linear Algebra and Calculus</td><td class="py-2.5 px-3">CBT Hall-B</td></tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      `;
    }

    function uploadMilestoneDeliverable(milestoneId) {
      const m = state.microProject.milestones.find(x => x.id === milestoneId);
      if (!m) return;
      m.status = "Submitted for Review";
      m.file = `M${milestoneId}_Validation_BTEC26O1022.pdf`;
      m.remark = "Uploaded by student — Pending evaluation by Dr. Abhishek Sharma (AS).";
      render();
      showToast(`Milestone ${milestoneId} deliverable uploaded to Dr. Abhishek Sharma (AS)!`, "success");
    }

    function renderStudentMicroProject() {
      const mp = state.microProject;
      return `
        <div class="space-y-6">
          <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
            <div class="flex flex-col lg:flex-row gap-6">
              <div class="w-full lg:w-72 shrink-0 rounded-2xl bg-gradient-to-br from-mits-navy via-indigo-950 to-slate-900 p-5 text-white flex flex-col justify-between border border-amber-400/30">
                <div class="flex items-center justify-between text-[11px] font-mono text-amber-300">
                  <span>MICRO-PROJECT-I LAB</span>
                  <span>${mp.courseCode}</span>
                </div>
                <svg class="w-full h-28 my-2" viewBox="0 0 240 100" fill="none">
                  <rect x="10" y="25" width="54" height="50" rx="8" fill="#1E3A8A" stroke="#F59E0B" stroke-width="1.5"/>
                  <text x="37" y="48" fill="#FDE68A" font-size="9" text-anchor="middle" font-weight="bold">ESP32-S3</text>
                  <text x="37" y="61" fill="#CBD5E1" font-size="7" text-anchor="middle">Edge Node</text>
                  <path d="M64 50H94" stroke="#34D399" stroke-width="2" stroke-dasharray="3 3"/>
                  <rect x="94" y="20" width="56" height="60" rx="10" fill="#0F172A" stroke="#34D399" stroke-width="1.5"/>
                  <text x="122" y="47" fill="#6EE7B7" font-size="9" text-anchor="middle" font-weight="bold">MQTT Broker</text>
                  <text x="122" y="60" fill="#94A3B8" font-size="7" text-anchor="middle">TLS Telemetry</text>
                  <path d="M150 50H178" stroke="#F59E0B" stroke-width="2"/>
                  <rect x="178" y="25" width="52" height="50" rx="8" fill="#1E3A8A" stroke="#60A5FA" stroke-width="1.5"/>
                  <text x="204" y="48" fill="#93C5FD" font-size="9" text-anchor="middle" font-weight="bold">SAMS AI</text>
                  <text x="204" y="61" fill="#CBD5E1" font-size="7" text-anchor="middle">Analytics</text>
                </svg>
                <div class="text-[11px] text-emerald-300 font-semibold">
                  <i class="fa-solid fa-calendar-check mr-1"></i> Expo: ${mp.expoDates}
                </div>
              </div>

              <div class="flex-1 space-y-4">
                <div class="flex flex-wrap items-center justify-between gap-2">
                  <div class="flex items-center gap-2">
                    <span class="px-3 py-1 rounded-full bg-indigo-100 text-indigo-900 text-xs font-bold">Track: ${mp.track} Track</span>
                    <span class="px-3 py-1 rounded-full bg-emerald-100 text-emerald-800 text-xs font-bold">
                      <i class="fa-solid fa-circle-check mr-1"></i> Supervisor Approved
                    </span>
                  </div>
                  <span class="text-xs font-semibold text-slate-600">Project Supervisor: <strong class="text-slate-900">${mp.supervisor}</strong></span>
                </div>

                <div>
                  <h2 class="text-xl font-extrabold text-slate-900">${mp.title}</h2>
                  <p class="text-xs text-slate-600 mt-1.5 leading-relaxed">${mp.abstract}</p>
                </div>

                <div class="p-3.5 rounded-xl bg-slate-50 border border-slate-200">
                  <div class="text-[11px] font-bold text-slate-500 uppercase mb-1.5">Project Squad Members</div>
                  <div class="flex flex-wrap gap-2">
                    ${mp.teamMembers.map(m => `<span class="text-xs bg-white border border-slate-200 px-2.5 py-1 rounded-lg font-medium text-slate-700">${m}</span>`).join('')}
                  </div>
                </div>
              </div>
            </div>

            <div class="mt-6 pt-6 border-t border-slate-200">
              <h3 class="text-base font-bold text-slate-900 mb-3">5-Milestone Execution & Faculty Grading Tracker (Wednesdays 11:00 AM–01:00 PM • M003)</h3>
              <div class="space-y-3">
                ${mp.milestones.map(m => `
                  <div class="p-4 rounded-2xl border ${m.status === 'Graded' ? 'border-emerald-200 bg-emerald-50/30' : m.status === 'Pending Upload' ? 'border-amber-300 bg-amber-50/40' : 'border-slate-200 bg-white'} flex flex-col md:flex-row md:items-center justify-between gap-4">
                    <div class="space-y-1">
                      <div class="flex flex-wrap items-center gap-2">
                        <span class="px-2.5 py-0.5 rounded-lg bg-mits-navy text-white text-xs font-bold">Milestone ${m.id}</span>
                        <span class="font-bold text-slate-900 text-sm">${m.title}</span>
                      </div>
                      <p class="text-xs text-slate-600">${m.remark}</p>
                      <div class="text-[11px] text-slate-400 font-mono">
                        Deadline: ${m.deadline} ${m.file ? `• File: <span class="text-indigo-700 font-semibold">${m.file}</span>` : ''}
                      </div>
                    </div>

                    <div class="flex items-center gap-3 shrink-0">
                      <div class="text-right">
                        <div class="text-xs font-extrabold ${m.status === 'Graded' ? 'text-emerald-700' : 'text-amber-700'}">${m.score}</div>
                        <div class="text-[11px] text-slate-500">${m.status}</div>
                      </div>
                      ${m.status === 'Pending Upload' ? `
                        <button onclick="uploadMilestoneDeliverable(${m.id})"
                          class="px-3.5 py-2 rounded-xl bg-amber-500 hover:bg-amber-600 text-mits-navy font-bold text-xs shadow-xs transition cursor-pointer">
                          <i class="fa-solid fa-cloud-arrow-up mr-1"></i> Upload Deliverable
                        </button>
                      ` : ''}
                    </div>
                  </div>
                `).join('')}
              </div>
            </div>
          </div>

          <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
            <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-4 border-b border-slate-200">
              <div>
                <span class="px-2.5 py-0.5 rounded-full bg-purple-100 text-purple-800 text-xs font-bold">Course 33261108 • Mandatory SGPA Credit</span>
                <h3 class="text-lg font-extrabold text-slate-900 mt-1">Semester Proficiency Evaluation Portal</h3>
                <p class="text-xs text-slate-500">10-minute comprehensive subject overview & conceptual synthesis presentation (Weekly Friday 09:00 AM–11:00 AM in Room M003).</p>
              </div>
              <div class="px-4 py-2.5 rounded-xl bg-purple-50 border border-purple-200 text-xs">
                <div class="text-purple-700 font-semibold">Final Evaluation Window</div>
                <div class="font-extrabold text-purple-950 text-sm">Nov 27–29, 2026</div>
              </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-4 pt-4 text-xs">
              <div class="p-4 rounded-xl bg-slate-50 border border-slate-200">
                <div class="text-slate-400 font-semibold">Course Faculty / Panel Chair</div>
                <div class="text-sm font-bold text-slate-900 mt-1">Dr. Namita Arya (NA)</div>
                <div class="text-slate-500 mt-0.5">Cross-Department Evaluation Panel-B</div>
              </div>
              <div class="p-4 rounded-xl bg-slate-50 border border-slate-200">
                <div class="text-slate-400 font-semibold">Final Presentation Slot & Venue</div>
                <div class="text-sm font-bold text-indigo-900 mt-1">Nov 28, 2026 • 11:20 AM</div>
                <div class="text-slate-500 mt-0.5">Room M003 / Seminar Hall-3 • Slot #08</div>
              </div>
              <div class="p-4 rounded-xl bg-slate-50 border border-slate-200">
                <div class="text-slate-400 font-semibold">Assigned Proficiency Topic</div>
                <div class="text-sm font-bold text-slate-900 mt-1">Digital Logic & Embedded Microcontroller Interfacing</div>
                <div class="text-slate-500 mt-0.5">10 Mins Talk + 3 Mins Panel Q&A</div>
              </div>
            </div>
          </div>
        </div>
      `;
    }

    function openFeeReceiptModal(title, amount, txnId, date, breakdown) {
      openModal(`
        <div class="p-6 space-y-4">
          <div class="flex items-center justify-between border-b border-slate-200 pb-4">
            <div class="flex items-center gap-3">
              ${getMitsCrestSVG("w-10 h-10")}
              <div>
                <h3 class="font-extrabold text-slate-900 text-base">MITS Gwalior – Official Fee Receipt</h3>
                <p class="text-xs text-slate-500">Deemed University • Session July – Dec 2026</p>
              </div>
            </div>
            <span class="px-3 py-1 rounded-full bg-emerald-100 text-emerald-800 text-xs font-extrabold">PAID</span>
          </div>
          <div class="grid grid-cols-2 gap-3 text-xs bg-slate-50 p-4 rounded-xl border border-slate-200">
            <div><span class="text-slate-400">Student Name:</span> <strong class="block text-slate-800">${state.studentProfile.name}</strong></div>
            <div><span class="text-slate-400">Enrollment No:</span> <strong class="block font-mono text-slate-800">${state.studentProfile.enrollment}</strong></div>
            <div><span class="text-slate-400">Receipt / Txn ID:</span> <strong class="block font-mono text-indigo-700">${txnId}</strong></div>
            <div><span class="text-slate-400">Payment Date:</span> <strong class="block text-slate-800">${date}</strong></div>
          </div>
          <div class="border border-slate-200 rounded-xl overflow-hidden text-xs">
            <div class="bg-slate-100 px-4 py-2 font-bold text-slate-700 flex justify-between">
              <span>${title}</span><span>Amount</span>
            </div>
            <div class="p-4 space-y-2">
              ${breakdown.map(b => `<div class="flex justify-between text-slate-600"><span>${b.label}</span><span class="font-mono font-semibold">${b.val}</span></div>`).join('')}
              <div class="pt-2 border-t border-slate-200 flex justify-between font-extrabold text-slate-900 text-sm">
                <span>Total Paid</span><span class="font-mono text-emerald-700">${amount}</span>
              </div>
            </div>
          </div>
          <div class="flex justify-end gap-2">
            <button onclick="closeModal()" class="px-4 py-2 rounded-xl border border-slate-300 text-xs font-semibold">Close</button>
            <button onclick="closeModal(); showToast('Fee Receipt (${txnId}) downloaded!', 'success');" class="px-4 py-2 rounded-xl bg-mits-navy text-white text-xs font-semibold">
              <i class="fa-solid fa-print mr-1"></i> Download / Print Receipt
            </button>
          </div>
        </div>
      `);
    }

    function openAdmitCardModal() {
      const p = state.studentProfile;
      openModal(`
        <div class="p-6 space-y-4">
          <div class="flex items-center justify-between border-b-2 border-mits-navy pb-4">
            <div class="flex items-center gap-3">
              ${getMitsCrestSVG("w-12 h-12")}
              <div>
                <div class="text-xs font-bold text-amber-700 uppercase">Controller of Examinations • MITS Gwalior</div>
                <h3 class="text-lg font-extrabold text-slate-900">End-Semester Hall Ticket / Admit Card</h3>
                <p class="text-xs text-slate-500">B.Tech 1st Semester (EACE – Centre for IoT) • Dec 2026</p>
              </div>
            </div>
            <div class="w-14 h-14 rounded-xl border border-slate-300 bg-slate-50 flex flex-col items-center justify-center text-[9px] font-mono text-slate-500">
              <i class="fa-solid fa-qrcode text-2xl text-slate-800 mb-0.5"></i>
              VERIFIED
            </div>
          </div>

          <div class="grid grid-cols-2 sm:grid-cols-3 gap-3 text-xs bg-slate-50 p-3.5 rounded-xl border border-slate-200">
            <div><span class="text-slate-400">Candidate:</span><strong class="block text-slate-900">${p.name}</strong></div>
            <div><span class="text-slate-400">Enrollment:</span><strong class="block font-mono text-indigo-800">${p.enrollment}</strong></div>
            <div><span class="text-slate-400">Father's Name:</span><strong class="block text-slate-900">${p.fatherName}</strong></div>
            <div><span class="text-slate-400">Branch:</span><strong class="block text-slate-900">EACE (CIOT)</strong></div>
            <div><span class="text-slate-400">Exam Center:</span><strong class="block text-slate-900">MITS CBT & CIOT Block</strong></div>
            <div><span class="text-slate-400">Eligibility:</span><strong class="block text-emerald-700">PROVISIONALLY ELIGIBLE</strong></div>
          </div>

          <div class="text-xs border border-slate-200 rounded-xl overflow-hidden">
            <table class="w-full text-left">
              <thead class="bg-mits-navy text-white">
                <tr><th class="py-2 px-3">Code</th><th class="py-2 px-3">Subject Title</th><th class="py-2 px-3">Exam Date</th></tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                <tr><td class="py-1.5 px-3 font-mono">33261101</td><td class="py-1.5 px-3">Introduction to Electrical and Computer Engineering</td><td class="py-1.5 px-3">Dec 10, 2026</td></tr>
                <tr><td class="py-1.5 px-3 font-mono">33261102</td><td class="py-1.5 px-3">Computer Programming</td><td class="py-1.5 px-3">Dec 12, 2026</td></tr>
                <tr><td class="py-1.5 px-3 font-mono">33261103</td><td class="py-1.5 px-3">Basics of Embedded Programming</td><td class="py-1.5 px-3">Dec 14, 2026</td></tr>
                <tr><td class="py-1.5 px-3 font-mono">33261104</td><td class="py-1.5 px-3">Digital Logic Design</td><td class="py-1.5 px-3">Dec 16, 2026</td></tr>
                <tr><td class="py-1.5 px-3 font-mono">33261105</td><td class="py-1.5 px-3">Linear Algebra and Calculus</td><td class="py-1.5 px-3">Dec 18, 2026</td></tr>
              </tbody>
            </table>
          </div>

          <div class="flex justify-end gap-2">
            <button onclick="closeModal()" class="px-4 py-2 rounded-xl border border-slate-300 text-xs font-semibold">Close</button>
            <button onclick="closeModal(); showToast('Hall Ticket PDF downloaded for ${p.enrollment}!', 'success');" class="px-4 py-2 rounded-xl bg-indigo-900 text-white text-xs font-semibold">
              <i class="fa-solid fa-download mr-1.5"></i> Download Hall Ticket
            </button>
          </div>
        </div>
      `);
    }

    function openDocumentGeneratorModal(purpose) {
      const p = state.studentProfile;
      let heading = "Official Bonafide Certificate (Scholarship Purpose)";
      let refCode = "MITS/CIOT/2026-27/SCH-904";
      let bodyParagraph = `
        This is to certify that <strong>Ms. ${p.name}</strong>, D/o <strong>${p.fatherName}</strong>, bearing Enrollment Number <strong>${p.enrollment}</strong>, is a regular bonafide student of <strong>B.Tech 1st Year (1st Semester) in Electrical and Computer Engineering (EACE)</strong> under the <strong>Centre for Internet of Things (CIOT)</strong> for the Academic Session July – Dec 2026.<br/><br/>
        She has deposited the prescribed Semester-1 Academic Fee of <strong>₹1,40,000</strong> (Tuition, Caution Deposit & Exam Fee). This Bonafide & Fee Verification Certificate is issued specifically for submission on the <strong>MP Post-Matric Scholarship / Mukhyamantri Medhavi Chhatra Yojana (MMVY) / NSP Portal</strong>.
      `;

      if (purpose === 'loan') {
        heading = "Bonafide & Official Fee Structure Letter (Education Loan Purpose)";
        refCode = "MITS/CIOT/2026-27/LOAN-319";
        bodyParagraph = `
          This is to certify that <strong>Ms. ${p.name}</strong>, D/o <strong>${p.fatherName}</strong> (Enrollment No: <strong>${p.enrollment}</strong>), is admitted to the 4-Year Full-Time <strong>B.Tech Degree Program in Electrical and Computer Engineering (EACE)</strong> at Madhav Institute of Technology & Science, Gwalior.<br/><br/>
          The annual institutional fee structure comprises <strong>₹1,40,000</strong> (Academic Tuition, Admission Deposit & Exam Fee) and <strong>₹78,000</strong> (Girls' Hostel No. 6 Accommodation & Mess Charges), totaling <strong>₹2,18,000</strong> per annum. This certificate is issued for Bank Education Loan sanction and disbursement.
        `;
      } else if (purpose === 'hostel') {
        heading = "Hostel Residency & General Bonafide Certificate";
        refCode = "MITS/HSTL/2026-27/GH6-112";
        bodyParagraph = `
          This is to certify that <strong>Ms. ${p.name}</strong>, D/o <strong>${p.fatherName}</strong> (Enrollment No: <strong>${p.enrollment}</strong>), B.Tech 1st Semester (EACE – CIOT), is a bonafide resident of <strong>${p.hostel}</strong> inside the MITS Gwalior Campus for the Academic Session 2026–27.
        `;
      }

      openModal(`
        <div class="p-6 space-y-4">
          <div class="flex items-center justify-between border-b border-slate-200 pb-3">
            <h3 class="font-extrabold text-slate-900 text-base">${heading}</h3>
            <button onclick="closeModal()" class="text-slate-400 hover:text-slate-700"><i class="fa-solid fa-xmark"></i></button>
          </div>
          <div class="p-5 rounded-2xl bg-amber-50/40 border border-amber-200 text-xs space-y-3 leading-relaxed text-slate-800">
            <div class="flex items-center justify-center gap-3 pb-2 border-b border-amber-200/70">
              ${getMitsCrestSVG("w-10 h-10")}
              <div class="text-left">
                <div class="font-extrabold text-mits-navy uppercase">Madhav Institute of Technology & Science, Gwalior</div>
                <div class="text-[11px] text-slate-500">Deemed University • Ref: ${refCode} • Date: ${CURRENT_DATE_STR}</div>
              </div>
            </div>
            <p>${bodyParagraph}</p>
            <div class="pt-4 flex justify-between items-end text-[11px] text-slate-500">
              <span class="font-mono"><i class="fa-solid fa-qrcode mr-1 text-slate-700"></i>Digitally Signed via MITS SAMS</span>
              <strong class="text-slate-900">Dean / Registrar, MITS Gwalior</strong>
            </div>
          </div>
          <div class="flex justify-end gap-2">
            <button onclick="closeModal()" class="px-4 py-2 rounded-xl border border-slate-300 text-xs font-semibold">Close</button>
            <button onclick="closeModal(); showToast('${heading} downloaded as PDF!', 'success');" class="px-4 py-2 rounded-xl bg-mits-navy text-white text-xs font-semibold cursor-pointer">
              <i class="fa-solid fa-file-arrow-down mr-1.5"></i> Download Signed PDF
            </button>
          </div>
        </div>
      `);
    }

    function raiseHelpDeskTicket(e) {
      e.preventDefault();
      const cat = document.getElementById('ticket-category').value;
      const subj = document.getElementById('ticket-subject').value.trim();
      if (!subj) return;
      state.tickets.unshift({
        id: `TKT-2026-${Math.floor(500 + Math.random() * 400)}`,
        category: cat,
        subject: subj,
        date: "Nov 17, 2026",
        status: "Open",
        reply: "Assigned to MITS Student Help Desk / Scholarship Nodal Officer. Expected response within 24 hrs."
      });
      render();
      showToast("Help Desk ticket raised successfully!", "success");
    }

    function renderStudentServices() {
      const tabs = [
        { id: 'fees', label: 'Academic & Hostel Fee Structure', icon: 'fa-indian-rupee-sign' },
        { id: 'scholarships', label: 'Ongoing Scholarships (MP / MMVY)', icon: 'fa-graduation-cap' },
        { id: 'bonafide', label: 'Bonafide (Scholarship & Loan)', icon: 'fa-certificate' },
        { id: 'admit-help', label: 'Admit Cards & Help Desk', icon: 'fa-headset' }
      ];

      return `
        <div class="space-y-6">
          <div class="bg-gradient-to-r from-mits-navy via-mits-royal to-indigo-900 rounded-2xl p-6 text-white shadow-md flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
              <span class="px-3 py-1 rounded-full bg-amber-400 text-mits-navy text-[11px] font-extrabold uppercase tracking-wider">
                Student Financial & Welfare Hub
              </span>
              <h2 class="text-xl sm:text-2xl font-extrabold mt-2">Fees, Scholarships, Bonafide & Help Desk</h2>
              <p class="text-xs text-slate-300 mt-1">
                Complete academic & hostel fee structure, live MP State / Central scholarship trackers, instant Scholarship/Loan Bonafide generator, and student support.
              </p>
            </div>
            <div class="flex flex-wrap gap-2">
              <div class="px-4 py-2.5 rounded-xl bg-white/10 border border-white/15 text-xs">
                <div class="text-slate-300">Sem 1 Total Paid</div>
                <div class="text-lg font-extrabold text-emerald-400">₹2,18,000</div>
              </div>
              <div class="px-4 py-2.5 rounded-xl bg-white/10 border border-white/15 text-xs">
                <div class="text-slate-300">Active Schemes</div>
                <div class="text-lg font-extrabold text-amber-300">${state.scholarships.length} Portals</div>
              </div>
            </div>
          </div>

          <div class="flex flex-wrap gap-2 bg-white p-2 rounded-2xl border border-slate-200 shadow-2xs">
            ${tabs.map(t => `
              <button onclick="state.servicesSubTab='${t.id}'; render();"
                class="px-4 py-2.5 rounded-xl text-xs font-bold transition cursor-pointer flex items-center gap-2 ${state.servicesSubTab === t.id ? 'bg-mits-navy text-white shadow-xs' : 'text-slate-600 hover:bg-slate-100'}">
                <i class="fa-solid ${t.icon} ${state.servicesSubTab === t.id ? 'text-amber-400' : 'text-indigo-600'}"></i>
                <span>${t.label}</span>
              </button>
            `).join('')}
          </div>

          ${state.servicesSubTab === 'fees' ? `
            <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-6">
              <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-slate-200 pb-4">
                <div>
                  <h3 class="text-lg font-extrabold text-slate-900">B.Tech (EACE – CIOT) Academic & Hostel Fee Structure</h3>
                  <p class="text-xs text-slate-500">Itemized fee ledger and downloadable digital receipts for ${state.studentProfile.name} (${state.studentProfile.enrollment}).</p>
                </div>
                <div class="flex bg-slate-100 p-1 rounded-xl self-start">
                  <button onclick="state.feeSubTab='sem1'; render();" class="px-4 py-1.5 rounded-lg text-xs font-bold cursor-pointer ${state.feeSubTab === 'sem1' ? 'bg-mits-navy text-white' : 'text-slate-600'}">
                    Semester 1 (Paid)
                  </button>
                  <button onclick="state.feeSubTab='sem2'; render();" class="px-4 py-1.5 rounded-lg text-xs font-bold cursor-pointer ${state.feeSubTab === 'sem2' ? 'bg-mits-navy text-white' : 'text-slate-600'}">
                    Semester 2 Structure
                  </button>
                </div>
              </div>

              ${state.feeSubTab === 'sem1' ? `
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                  <div class="rounded-2xl border border-emerald-200 bg-emerald-50/30 p-5 flex flex-col justify-between space-y-4">
                    <div class="space-y-3">
                      <div class="flex items-center justify-between">
                        <span class="px-2.5 py-0.5 rounded bg-indigo-100 text-indigo-900 text-xs font-bold">Institutional Academic Fee</span>
                        <span class="px-2.5 py-0.5 rounded-full bg-emerald-600 text-white text-[11px] font-bold">✓ PAID</span>
                      </div>
                      <div class="flex items-baseline justify-between">
                        <div class="text-3xl font-extrabold text-slate-900">₹1,40,000</div>
                        <span class="text-xs font-mono text-slate-500">Txn: MITS-ACAD-260814</span>
                      </div>
                      <div class="bg-white rounded-xl border border-slate-200 divide-y divide-slate-100 text-xs">
                        <div class="p-2.5 flex justify-between"><span class="text-slate-600">Tuition & CIOT Laboratory Fee</span><span class="font-mono font-bold">₹1,22,500</span></div>
                        <div class="p-2.5 flex justify-between"><span class="text-slate-600">One-Time Admission & Caution Deposit</span><span class="font-mono font-bold">₹12,000</span></div>
                        <div class="p-2.5 flex justify-between"><span class="text-slate-600">End-Sem Exam, Library & Sports Fee</span><span class="font-mono font-bold">₹5,500</span></div>
                      </div>
                    </div>
                    <button onclick='openFeeReceiptModal("Academic Fees – Semester 1", "₹1,40,000", "MITS-ACAD-260814", "Aug 14, 2026", [{"label":"Tuition & CIOT Laboratory Fee","val":"₹1,22,500"},{"label":"One-Time Admission & Caution Deposit","val":"₹12,000"},{"label":"End-Sem Exam, Library & Sports Fee","val":"₹5,500"}])'
                      class="w-full py-2.5 rounded-xl bg-mits-navy hover:bg-indigo-900 text-white text-xs font-bold transition cursor-pointer">
                      <i class="fa-solid fa-receipt mr-1.5 text-amber-400"></i> Download Academic Fee Receipt
                    </button>
                  </div>

                  <div class="rounded-2xl border border-emerald-200 bg-emerald-50/30 p-5 flex flex-col justify-between space-y-4">
                    <div class="space-y-3">
                      <div class="flex items-center justify-between">
                        <span class="px-2.5 py-0.5 rounded bg-amber-100 text-amber-900 text-xs font-bold">Hostel & Mess Fee (Girls' Hostel No. 6)</span>
                        <span class="px-2.5 py-0.5 rounded-full bg-emerald-600 text-white text-[11px] font-bold">✓ PAID</span>
                      </div>
                      <div class="flex items-baseline justify-between">
                        <div class="text-3xl font-extrabold text-slate-900">₹78,000</div>
                        <span class="text-xs font-mono text-slate-500">Txn: MITS-HSTL-260816</span>
                      </div>
                      <div class="bg-white rounded-xl border border-slate-200 divide-y divide-slate-100 text-xs">
                        <div class="p-2.5 flex justify-between"><span class="text-slate-600">Girls' Hostel No. 6 Room & Wi-Fi Charges</span><span class="font-mono font-bold">₹42,000</span></div>
                        <div class="p-2.5 flex justify-between"><span class="text-slate-600">Semester Central Dining & Mess Advance</span><span class="font-mono font-bold">₹31,000</span></div>
                        <div class="p-2.5 flex justify-between"><span class="text-slate-600">Hostel Security Deposit (Refundable)</span><span class="font-mono font-bold">₹5,000</span></div>
                      </div>
                    </div>
                    <button onclick='openFeeReceiptModal("Hostel & Mess Fees – Semester 1", "₹78,000", "MITS-HSTL-260816", "Aug 16, 2026", [{"label":"Girls Hostel No. 6 Room & Wi-Fi Charges","val":"₹42,000"},{"label":"Semester Central Dining & Mess Advance","val":"₹31,000"},{"label":"Hostel Security Deposit (Refundable)","val":"₹5,000"}])'
                      class="w-full py-2.5 rounded-xl bg-mits-navy hover:bg-indigo-900 text-white text-xs font-bold transition cursor-pointer">
                      <i class="fa-solid fa-receipt mr-1.5 text-amber-400"></i> Download Hostel & Mess Receipt
                    </button>
                  </div>
                </div>
              ` : `
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                  <div class="p-5 rounded-2xl border border-slate-200 bg-slate-50 space-y-3 text-xs">
                    <div class="flex justify-between items-center">
                      <span class="font-bold text-slate-800 text-sm">Semester 2 Academic Fee (Jan – May 2027)</span>
                      <span class="px-2.5 py-0.5 rounded bg-amber-100 text-amber-900 font-bold">Opens Jan 2027</span>
                    </div>
                    <div class="text-2xl font-extrabold text-slate-900">₹1,28,000 <span class="text-xs font-normal text-slate-500">(Excludes one-time admission deposit)</span></div>
                    <p class="text-slate-500">Includes Semester 2 Tuition, CIOT Lab Fee & Examination Fee.</p>
                  </div>
                  <div class="p-5 rounded-2xl border border-slate-200 bg-slate-50 space-y-3 text-xs">
                    <div class="flex justify-between items-center">
                      <span class="font-bold text-slate-800 text-sm">Semester 2 Hostel & Mess Fee</span>
                      <span class="px-2.5 py-0.5 rounded bg-amber-100 text-amber-900 font-bold">Opens Jan 2027</span>
                    </div>
                    <div class="text-2xl font-extrabold text-slate-900">₹73,000 <span class="text-xs font-normal text-slate-500">(Excludes one-time hostel security)</span></div>
                    <p class="text-slate-500">Includes Girls' Hostel No. 6 accommodation & Semester 2 mess charges.</p>
                  </div>
                </div>
              `}
            </div>
          ` : ''}

          ${state.servicesSubTab === 'scholarships' ? `
            <div class="space-y-5">
              <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                <div>
                  <h3 class="text-lg font-extrabold text-slate-900">Ongoing State & National Scholarship Schemes (2026–27)</h3>
                  <p class="text-xs text-slate-500 mt-0.5">Track your MP Post-Matric Scholarship, Mukhyamantri Medhavi Chhatra Yojana (MMVY) & AICTE/NSP applications.</p>
                </div>
                <button onclick="state.servicesSubTab='bonafide'; render();" class="px-4 py-2.5 rounded-xl bg-amber-500 hover:bg-amber-600 text-mits-navy text-xs font-extrabold cursor-pointer shrink-0">
                  <i class="fa-solid fa-certificate mr-1.5"></i> Get Scholarship Bonafide →
                </button>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                ${state.scholarships.map(sch => `
                  <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs flex flex-col justify-between space-y-4 hover:border-indigo-400 transition">
                    <div class="space-y-2.5">
                      <div class="flex items-center justify-between gap-2">
                        <span class="px-2.5 py-0.5 rounded-full bg-emerald-100 text-emerald-800 text-[11px] font-bold">${sch.status}</span>
                        <span class="text-[11px] font-mono text-slate-500">Ref: ${sch.appId}</span>
                      </div>
                      <h4 class="font-extrabold text-slate-900 text-base">${sch.name}</h4>
                      <p class="text-[11px] font-semibold text-indigo-800">${sch.authority}</p>
                      <div class="p-3 rounded-xl bg-amber-50/70 border border-amber-200 text-xs text-amber-950">
                        <strong>Benefit:</strong> ${sch.benefit}
                      </div>
                      <p class="text-xs text-slate-600 leading-relaxed"><strong>Eligibility:</strong> ${sch.eligibility}</p>
                    </div>
                    <div class="pt-3 border-t border-slate-100 flex items-center justify-between text-xs">
                      <span class="text-slate-500">Portal Deadline: <strong class="text-rose-700">${sch.deadline}</strong></span>
                      <button onclick="openDocumentGeneratorModal('scholarship')" class="px-3 py-1.5 rounded-xl bg-indigo-50 hover:bg-indigo-900 text-indigo-900 hover:text-white font-bold transition cursor-pointer">
                        Download Attested Bonafide
                      </button>
                    </div>
                  </div>
                `).join('')}
              </div>
            </div>
          ` : ''}

          ${state.servicesSubTab === 'bonafide' ? `
            <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-5">
              <div>
                <h3 class="text-lg font-extrabold text-slate-900">Instant Digital Bonafide & Official Certificate Desk</h3>
                <p class="text-xs text-slate-500 mt-0.5">
                  Pre-filled with verified records for <strong>${state.studentProfile.name}</strong> (D/o <strong>${state.studentProfile.fatherName}</strong>, ${state.studentProfile.enrollment}). Choose the exact certificate format required:
                </p>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-3 gap-5">
                <div class="p-5 rounded-2xl border-2 border-indigo-200 bg-indigo-50/30 flex flex-col justify-between space-y-4">
                  <div class="space-y-2">
                    <div class="w-11 h-11 rounded-xl bg-indigo-900 text-amber-400 flex items-center justify-center text-lg">
                      <i class="fa-solid fa-graduation-cap"></i>
                    </div>
                    <h4 class="font-extrabold text-slate-900 text-base">Bonafide for Scholarship Purpose</h4>
                    <p class="text-xs text-slate-600 leading-relaxed">
                      Tailored for <strong>MP Post-Matric Scholarship</strong>, <strong>MMVY</strong>, and <strong>NSP</strong> portals. Includes Semester-1 Paid Tuition Fee attestation.
                    </p>
                  </div>
                  <button onclick="openDocumentGeneratorModal('scholarship')"
                    class="w-full py-2.5 rounded-xl bg-indigo-900 hover:bg-indigo-800 text-white text-xs font-bold cursor-pointer">
                    <i class="fa-solid fa-file-pdf mr-1.5"></i> Preview & Download PDF
                  </button>
                </div>

                <div class="p-5 rounded-2xl border-2 border-amber-200 bg-amber-50/30 flex flex-col justify-between space-y-4">
                  <div class="space-y-2">
                    <div class="w-11 h-11 rounded-xl bg-amber-500 text-mits-navy flex items-center justify-center text-lg">
                      <i class="fa-solid fa-building-columns"></i>
                    </div>
                    <h4 class="font-extrabold text-slate-900 text-base">Bonafide & Fee Letter for Education Loan</h4>
                    <p class="text-xs text-slate-600 leading-relaxed">
                      Tailored for <strong>SBI / Bank Education Loan</strong> processing. Certifies 4-year B.Tech admission along with ₹1,40,000 Academic + ₹78,000 Hostel fee schedule.
                    </p>
                  </div>
                  <button onclick="openDocumentGeneratorModal('loan')"
                    class="w-full py-2.5 rounded-xl bg-amber-500 hover:bg-amber-600 text-mits-navy text-xs font-extrabold cursor-pointer">
                    <i class="fa-solid fa-file-pdf mr-1.5"></i> Preview & Download PDF
                  </button>
                </div>

                <div class="p-5 rounded-2xl border border-slate-200 bg-slate-50 flex flex-col justify-between space-y-4">
                  <div class="space-y-2">
                    <div class="w-11 h-11 rounded-xl bg-slate-800 text-white flex items-center justify-center text-lg">
                      <i class="fa-solid fa-house-user"></i>
                    </div>
                    <h4 class="font-extrabold text-slate-900 text-base">Hostel & General Bonafide</h4>
                    <p class="text-xs text-slate-600 leading-relaxed">
                      Certifies campus residency at <strong>Girls' Hostel No. 6</strong> and regular enrollment for Passport, Bank KYC, or Internship applications.
                    </p>
                  </div>
                  <button onclick="openDocumentGeneratorModal('hostel')"
                    class="w-full py-2.5 rounded-xl bg-slate-900 hover:bg-slate-800 text-white text-xs font-bold cursor-pointer">
                    <i class="fa-solid fa-file-pdf mr-1.5"></i> Preview & Download PDF
                  </button>
                </div>
              </div>
            </div>
          ` : ''}

          ${state.servicesSubTab === 'admit-help' ? `
            <div class="space-y-6">
              <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                <div class="flex items-center gap-4">
                  <div class="w-12 h-12 rounded-2xl bg-indigo-100 text-indigo-800 flex items-center justify-center text-xl shrink-0">
                    <i class="fa-solid fa-id-card-clip"></i>
                  </div>
                  <div>
                    <h3 class="font-extrabold text-slate-900 text-base">End-Semester Examination Hall Ticket (Dec 2026)</h3>
                    <p class="text-xs text-slate-500">Digitally signed Admit Card for B.Tech 1st Semester EACE (CBT & Lab Major Exams).</p>
                  </div>
                </div>
                <button onclick="openAdmitCardModal()" class="px-5 py-2.5 rounded-xl bg-indigo-900 hover:bg-indigo-800 text-white text-xs font-bold cursor-pointer shrink-0">
                  <i class="fa-solid fa-download mr-1.5"></i> Preview & Download Hall Ticket
                </button>
              </div>

              <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs">
                <h3 class="text-base font-extrabold text-slate-900 mb-4">MITS Student Help Desk (Scholarship, Fee & Hostel Queries)</h3>
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                  <form onsubmit="raiseHelpDeskTicket(event)" class="space-y-3 bg-slate-50 p-4 rounded-2xl border border-slate-200">
                    <div class="text-xs font-bold text-slate-800 uppercase">Raise New Support Ticket</div>
                    <div>
                      <label class="block text-xs font-semibold text-slate-600 mb-1">Issue Category</label>
                      <select id="ticket-category" class="w-full px-3 py-2 rounded-xl border border-slate-300 bg-white text-xs font-semibold">
                        <option value="Scholarship">Scholarship (MP Post-Matric / MMVY / NSP)</option>
                        <option value="Fee & Accounts">Fee Receipt & Accounts Section</option>
                        <option value="Hostel & Mess">Girls' Hostel No. 6 & Mess Office</option>
                        <option value="Academic">Academic / CIOT Office</option>
                      </select>
                    </div>
                    <div>
                      <label class="block text-xs font-semibold text-slate-600 mb-1">Query Subject & Details</label>
                      <textarea id="ticket-subject" rows="3" required placeholder="Describe your scholarship, fee, or hostel query..."
                        class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs focus:ring-2 focus:ring-indigo-500"></textarea>
                    </div>
                    <button type="submit" class="w-full py-2.5 rounded-xl bg-mits-navy hover:bg-slate-800 text-white text-xs font-bold cursor-pointer">
                      Submit Help Desk Ticket
                    </button>
                  </form>

                  <div class="lg:col-span-2 space-y-3">
                    ${state.tickets.map(t => `
                      <div class="p-4 rounded-2xl border border-slate-200 bg-white space-y-1.5">
                        <div class="flex flex-wrap items-center justify-between gap-2">
                          <div class="flex items-center gap-2">
                            <span class="font-mono text-xs font-bold text-indigo-800">${t.id}</span>
                            <span class="px-2 py-0.5 rounded bg-slate-100 text-slate-700 text-[11px] font-semibold">${t.category}</span>
                          </div>
                          <span class="px-2.5 py-0.5 rounded-full text-[11px] font-bold ${t.status === 'Resolved' ? 'bg-emerald-100 text-emerald-800' : 'bg-amber-100 text-amber-800'}">
                            ${t.status}
                          </span>
                        </div>
                        <div class="font-bold text-slate-900 text-sm">${t.subject}</div>
                        <p class="text-xs text-slate-600 bg-slate-50 p-2.5 rounded-xl border border-slate-100">
                          <strong>Admin Response:</strong> ${t.reply}
                        </p>
                      </div>
                    `).join('')}
                  </div>
                </div>
              </div>
            </div>
          ` : ''}
        </div>
      `;
    }

    function registerForCampusEvent(eventId) {
      const evt = state.events.find(e => e.id === eventId);
      if (!evt) return;
      openModal(`
        <div class="p-6 space-y-4">
          <div class="flex items-start justify-between border-b border-slate-200 pb-3">
            <div>
              <span class="px-2.5 py-0.5 rounded-full bg-indigo-100 text-indigo-800 text-xs font-bold">${evt.club}</span>
              <h3 class="text-lg font-extrabold text-slate-900 mt-1">Register for ${evt.title}</h3>
              <p class="text-xs text-slate-500">${evt.date} • Venue: ${evt.venue}</p>
            </div>
            <button onclick="closeModal()" class="text-slate-400 hover:text-slate-700"><i class="fa-solid fa-xmark"></i></button>
          </div>
          <div class="space-y-3 text-xs">
            <div class="p-3 rounded-xl bg-slate-50 border border-slate-200">
              <div><strong>Student:</strong> ${state.studentProfile.name} (${state.studentProfile.enrollment})</div>
              <div class="mt-0.5"><strong>Branch:</strong> EACE 1st Sem (CIOT)</div>
            </div>
            <div>
              <label class="block font-bold text-slate-700 mb-1">Select Participation Tier</label>
              <select class="w-full px-3 py-2.5 rounded-xl border border-slate-300 bg-white font-semibold">
                <option>Solo Entry — ${evt.fees.solo}</option>
                ${evt.fees.duo !== 'N/A' ? `<option>Duo Entry — ${evt.fees.duo}</option>` : ''}
                ${evt.fees.squad !== 'N/A' ? `<option>Squad / Team Entry — ${evt.fees.squad}</option>` : ''}
              </select>
            </div>
            <div class="p-3 rounded-xl bg-amber-50 border border-amber-200 text-amber-900">
              <strong>Perks Included:</strong> ${evt.perks}
            </div>
          </div>
          <div class="flex justify-end gap-2 pt-2">
            <button onclick="closeModal()" class="px-4 py-2 rounded-xl border border-slate-300 text-xs font-semibold">Cancel</button>
            <button onclick="confirmEventReg('${evt.id}')" class="px-5 py-2 rounded-xl bg-emerald-700 hover:bg-emerald-600 text-white text-xs font-bold cursor-pointer">
              Confirm Instant Registration
            </button>
          </div>
        </div>
      `);
    }

    function confirmEventReg(eventId) {
      const evt = state.events.find(e => e.id === eventId);
      if (evt) {
        evt.registered = true;
        evt.registrations += 1;
      }
      closeModal();
      render();
      showToast(`Registered for ${evt.title}! E-Pass added to your profile.`, "success");
    }

    function renderStudentClubs() {
      const categories = ['All', 'Tech & Societies', 'Cultural & Literary', 'Professional, Sports & Cells'];
      const filteredClubs = state.clubCategoryFilter === 'All'
        ? state.clubsDirectory
        : state.clubsDirectory.filter(c => c.category === state.clubCategoryFilter);

      return `
        <div class="space-y-8">
          <div class="space-y-4">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
              <div>
                <h2 class="text-xl font-extrabold text-slate-900">MITS Campus Events, Hackathons & Workshops</h2>
                <p class="text-xs text-slate-500">Live feed synced directly with MITS Club & Society Council.</p>
              </div>
              <span class="px-3 py-1 rounded-full bg-emerald-100 text-emerald-800 text-xs font-bold self-start">
                ${state.events.length} Active Campus Events
              </span>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
              ${state.events.map(evt => `
                <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-xs hover:border-indigo-400 transition flex flex-col justify-between space-y-4">
                  <div class="space-y-2.5">
                    <div class="flex items-center justify-between gap-2">
                      <span class="px-2.5 py-0.5 rounded-lg bg-indigo-50 text-indigo-800 text-xs font-bold">${evt.club}</span>
                      <span class="text-[11px] font-semibold text-slate-500"><i class="fa-solid fa-users mr-1"></i>${evt.registrations} Regs</span>
                    </div>
                    <h3 class="text-lg font-extrabold text-slate-900">${evt.title}</h3>
                    <p class="text-xs text-slate-600 leading-relaxed">${evt.desc}</p>

                    <div class="grid grid-cols-2 gap-2 text-xs pt-1">
                      <div class="p-2.5 rounded-xl bg-slate-50 border border-slate-100">
                        <div class="text-[10px] text-slate-400 uppercase font-bold">Date & Time</div>
                        <div class="font-semibold text-slate-800 mt-0.5">${evt.date}</div>
                      </div>
                      <div class="p-2.5 rounded-xl bg-slate-50 border border-slate-100">
                        <div class="text-[10px] text-slate-400 uppercase font-bold">Campus Venue</div>
                        <div class="font-semibold text-amber-800 mt-0.5"><i class="fa-solid fa-location-dot mr-1"></i>${evt.venue}</div>
                      </div>
                    </div>

                    <div class="p-3 rounded-xl bg-amber-50/60 border border-amber-200/80 text-xs space-y-1">
                      <div class="font-bold text-slate-800">
                        Entry Tiers: <span class="font-mono font-semibold text-indigo-900">Solo: ${evt.fees.solo} • Duo: ${evt.fees.duo} • Squad: ${evt.fees.squad}</span>
                      </div>
                      <div class="text-amber-900"><i class="fa-solid fa-award text-amber-600 mr-1"></i><strong>Perks:</strong> ${evt.perks}</div>
                    </div>
                  </div>

                  <div class="pt-2 border-t border-slate-100 flex items-center justify-between">
                    <span class="text-[11px] text-slate-400">${evt.category}</span>
                    ${evt.registered ? `
                      <span class="px-4 py-2 rounded-xl bg-emerald-100 text-emerald-800 text-xs font-extrabold">
                        <i class="fa-solid fa-circle-check mr-1"></i> Registered
                      </span>
                    ` : `
                      <button onclick="registerForCampusEvent('${evt.id}')"
                        class="px-4 py-2 rounded-xl bg-mits-navy hover:bg-indigo-900 text-white text-xs font-bold transition cursor-pointer">
                        Register Now →
                      </button>
                    `}
                  </div>
                </div>
              `).join('')}
            </div>
          </div>

          <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-5">
            <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
              <div>
                <h3 class="text-lg font-extrabold text-slate-900">MITS Gwalior Clubs, Technical Societies & Cells Directory</h3>
                <p class="text-xs text-slate-500">Explore student-led chapters across technology, arts, sports, and student welfare.</p>
              </div>
              <div class="flex flex-wrap gap-1.5">
                ${categories.map(cat => `
                  <button onclick="state.clubCategoryFilter='${cat}'; render();"
                    class="px-3 py-1.5 rounded-xl text-xs font-semibold transition cursor-pointer ${state.clubCategoryFilter === cat ? 'bg-mits-navy text-white' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}">
                    ${cat}
                  </button>
                `).join('')}
              </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
              ${filteredClubs.map(club => `
                <div class="p-4 rounded-2xl border border-slate-200 bg-slate-50/50 hover:bg-white hover:shadow-sm transition space-y-2">
                  <span class="inline-block text-[10px] font-bold uppercase px-2 py-0.5 rounded bg-indigo-100 text-indigo-800">${club.category}</span>
                  <h4 class="font-bold text-slate-900 text-sm">${club.name}</h4>
                  <p class="text-xs text-slate-600 leading-relaxed">${club.desc}</p>
                  <div class="pt-2 border-t border-slate-200/70 text-[11px] text-slate-500 flex justify-between">
                    <span>Lead: <strong class="text-slate-700">${club.lead}</strong></span>
                    <span>Mentor: ${club.faculty}</span>
                  </div>
                </div>
              `).join('')}
            </div>
          </div>
        </div>
      `;
    }

    function toggleFacultyStudentAttendance(roll) {
      const stu = state.facultyRoster.find(s => s.roll === roll);
      if (stu) stu.present = !stu.present;
      render();
    }

    function markFilteredFacultyAttendance(val) {
      state.facultyRoster.forEach(s => {
        if (state.facultyCohortMode === 'all80' ||
            (state.facultyCohortMode === 'batchA' && s.batch === 'Batch A') ||
            (state.facultyCohortMode === 'batchB' && s.batch === 'Batch B')) {
          s.present = val;
        }
      });
      render();
    }

    function saveFacultyAttendanceToERP() {
      const courseCode = state.facultyProfile.selectedCourse;
      const course = state.courses.find(c => c.code === courseCode) || state.courses[1];
      const ashi = state.facultyRoster.find(s => s.roll === 'BTEC26O1022');

      if (course && ashi) {
        course.total += 1;
        if (ashi.present) course.attended += 1;
      }
      render();
      showToast(`Attendance committed for ${course.shortName}! Ashi Golhani's attendance is now ${course.attended}/${course.total}.`, "success");
    }

    function addDraftQuizQuestion(e) {
      e.preventDefault();
      const qType = document.getElementById('fq-type').value;
      const qText = document.getElementById('fq-text').value.trim();
      const qPts = parseInt(document.getElementById('fq-points').value, 10) || 2;
      const opt1 = document.getElementById('fq-opt1').value.trim();
      const opt2 = document.getElementById('fq-opt2').value.trim();
      const opt3 = document.getElementById('fq-opt3').value.trim();
      const opt4 = document.getElementById('fq-opt4').value.trim();
      const correctVal = document.getElementById('fq-correct').value.trim();

      if (!qText) return;

      if (qType === 'integer') {
        state.facultyQuizDraftQuestions.push({
          type: 'integer',
          points: qPts,
          text: qText,
          correct: parseInt(correctVal, 10) || 0
        });
      } else {
        const opts = [opt1 || "Option A", opt2 || "Option B", opt3 || "Option C", opt4 || "Option D"];
        const cIdx = Math.max(0, Math.min(3, (parseInt(correctVal, 10) || 1) - 1));
        state.facultyQuizDraftQuestions.push({
          type: 'mcq',
          points: qPts,
          text: qText,
          options: opts,
          correctIndex: cIdx
        });
      }

      document.getElementById('fq-text').value = "";
      render();
      showToast("Question added to quiz draft!", "info");
    }

    function publishFacultyQuizToStudents(e) {
      e.preventDefault();
      const cCode = document.getElementById('fq-course').value;
      const qTitle = document.getElementById('fq-title').value.trim();
      const qMins = parseInt(document.getElementById('fq-duration').value, 10) || 10;

      if (!qTitle || state.facultyQuizDraftQuestions.length === 0) {
        showToast("Please enter a quiz title and at least 1 question.", "warning");
        return;
      }

      const course = state.courses.find(c => c.code === cCode) || state.courses[1];
      const totalPts = state.facultyQuizDraftQuestions.reduce((sum, item) => sum + (item.points || 2), 0);

      const newQuiz = {
        id: `QZ-${cCode}-${Date.now()}`,
        title: qTitle,
        courseCode: course.code,
        courseName: course.name,
        faculty: state.facultyProfile.name,
        durationMins: qMins,
        totalMarks: totalPts,
        status: "not_started",
        strikes: 0,
        timeLeft: qMins * 60,
        score: 0,
        grade: "A+",
        percentile: "95.0%ile",
        answers: {},
        questions: structuredClone(state.facultyQuizDraftQuestions)
      };

      state.quizzes.unshift(newQuiz);
      state.activeQuizIndex = 0;
      state.announcements.unshift(`⚡ NEW LIVE QUIZ PUBLISHED: "${qTitle}" (${course.code} – ${course.shortName}) by ${state.facultyProfile.name}!`);
      render();
      showToast(`"${qTitle}" published live! Students can now attempt it in Live In-Class Quizzes.`, "success");
    }

    function updateAshiTierMark(courseCode, tier, val) {
      const c = state.courses.find(x => x.code === courseCode);
      if (!c) return;
      const num = val === "" ? null : Number(val);
      if (tier === 'classQuiz') {
        c.classQuizMarks = num ?? 0;
      } else if (tier === 'centralQuiz') {
        c.centralQuiz = num;
      } else if (tier === 'minor') {
        c.minorMarks = num;
        c.minorStatus = num !== null ? `Evaluated (${num}/30)` : "Result Pending";
      } else if (tier === 'major') {
        c.majorMarks = num;
        c.majorStatus = num !== null ? `Published (${num}/50)` : "Scheduled Dec 2026";
      }
      saveState();
    }

    function uploadFacultyMaterial(e) {
      e.preventDefault();
      const code = document.getElementById('fac-mat-course').value;
      const unit = parseInt(document.getElementById('fac-mat-unit').value, 10);
      const type = document.getElementById('fac-mat-type').value;
      const title = document.getElementById('fac-mat-title').value.trim();
      if (!title) return;

      const course = state.courses.find(c => c.code === code);
      if (course) {
        course.materials.unshift({
          unit,
          title,
          type,
          size: "2.1 MB",
          date: "Nov 17, 2026",
          by: state.facultyProfile.name
        });
      }
      document.getElementById('fac-mat-title').value = "";
      render();
      showToast(`Published "${title}" to Student Syllabus Repository (${code})!`, "success");
    }

    function renderFacultyPortal() {
      const fp = state.facultyProfile;
      const filteredRoster = state.facultyRoster.filter(s => {
        if (state.facultyCohortMode === 'batchA') return s.batch === 'Batch A';
        if (state.facultyCohortMode === 'batchB') return s.batch === 'Batch B';
        return true;
      });
      const presentSample = filteredRoster.filter(s => s.present).length;
      const cohortDisplayTotal = state.facultyCohortMode === 'all80' ? 80 : 40;

      return `
        <div class="min-h-screen bg-slate-100 flex flex-col">
          <header class="bg-mits-navy text-white px-4 sm:px-8 py-4 shadow-lg border-b border-amber-500/30">
            <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-4">
              <div class="flex items-center gap-3.5">
                ${getMitsCrestSVG("w-11 h-11")}
                <div>
                  <div class="flex items-center gap-2">
                    <span class="px-2 py-0.5 rounded bg-amber-400 text-mits-navy text-[10px] font-extrabold uppercase">Faculty ERP</span>
                    <span class="text-xs text-slate-300 font-mono">${CURRENT_DATE_STR}</span>
                  </div>
                  <h1 class="text-lg font-extrabold mt-0.5">${fp.name}</h1>
                  <p class="text-xs text-amber-300">${fp.designation} • ${fp.department}</p>
                </div>
              </div>

              <div class="flex flex-wrap items-center gap-3">
                <select onchange="state.facultyProfile.selectedCourse=this.value; render();"
                  class="px-3 py-2 rounded-xl bg-white/10 border border-white/20 text-xs font-semibold text-white">
                  ${state.courses.map(c => `<option class="text-slate-900" value="${c.code}" ${fp.selectedCourse === c.code ? 'selected' : ''}>${c.code} – ${c.shortName} (${c.type})</option>`).join('')}
                </select>
                <button onclick="logoutToLanding()" class="px-4 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white text-xs font-bold cursor-pointer">
                  <i class="fa-solid fa-right-from-bracket mr-1"></i> Switch Portal
                </button>
              </div>
            </div>
          </header>

          <main class="flex-1 max-w-7xl w-full mx-auto p-4 sm:p-6 space-y-6">
            <div class="flex flex-wrap gap-2 bg-white p-2 rounded-2xl border border-slate-200 shadow-2xs">
              <button onclick="state.facultyTab='attendance'; render();"
                class="px-4 py-2.5 rounded-xl text-xs font-bold transition cursor-pointer ${state.facultyTab === 'attendance' ? 'bg-mits-navy text-white' : 'text-slate-600 hover:bg-slate-100'}">
                <i class="fa-solid fa-clipboard-user mr-1.5"></i> 1. EACE Class Attendance
              </button>
              <button onclick="state.facultyTab='quiz-builder'; render();"
                class="px-4 py-2.5 rounded-xl text-xs font-bold transition cursor-pointer ${state.facultyTab === 'quiz-builder' ? 'bg-mits-navy text-white' : 'text-slate-600 hover:bg-slate-100'}">
                <i class="fa-solid fa-bolt mr-1.5 text-amber-500"></i> 2. Create & Publish Live Quiz
              </button>
              <button onclick="state.facultyTab='marks'; render();"
                class="px-4 py-2.5 rounded-xl text-xs font-bold transition cursor-pointer ${state.facultyTab === 'marks' ? 'bg-mits-navy text-white' : 'text-slate-600 hover:bg-slate-100'}">
                <i class="fa-solid fa-pen-to-square mr-1.5"></i> 3. 4-Tier Exam Marks Entry
              </button>
              <button onclick="state.facultyTab='materials'; render();"
                class="px-4 py-2.5 rounded-xl text-xs font-bold transition cursor-pointer ${state.facultyTab === 'materials' ? 'bg-mits-navy text-white' : 'text-slate-600 hover:bg-slate-100'}">
                <i class="fa-solid fa-file-arrow-up mr-1.5"></i> 4. Upload Unit PDFs
              </button>
            </div>

            ${state.facultyTab === 'attendance' ? `
              <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-5">
                <div class="p-3.5 rounded-xl bg-indigo-50/80 border border-indigo-200 text-xs text-indigo-950 flex flex-wrap items-center justify-between gap-3">
                  <div>
                    <strong>EACE Cohort Structure:</strong> All <strong>80 students</strong> attend Theory lectures together in Room M003. For practical sessions only, the class divides into <strong>Batch A (40 students)</strong> and <strong>Batch B (40 students)</strong> due to lab workstation capacity.
                  </div>
                  <div class="flex bg-white border border-indigo-200 p-1 rounded-xl shrink-0">
                    <button onclick="state.facultyCohortMode='all80'; render();" class="px-3 py-1 rounded-lg text-xs font-bold cursor-pointer ${state.facultyCohortMode === 'all80' ? 'bg-indigo-900 text-white' : 'text-slate-600'}">
                      Theory: All 80
                    </button>
                    <button onclick="state.facultyCohortMode='batchA'; render();" class="px-3 py-1 rounded-lg text-xs font-bold cursor-pointer ${state.facultyCohortMode === 'batchA' ? 'bg-indigo-900 text-white' : 'text-slate-600'}">
                      Lab Batch A (40)
                    </button>
                    <button onclick="state.facultyCohortMode='batchB'; render();" class="px-3 py-1 rounded-lg text-xs font-bold cursor-pointer ${state.facultyCohortMode === 'batchB' ? 'bg-indigo-900 text-white' : 'text-slate-600'}">
                      Lab Batch B (40)
                    </button>
                  </div>
                </div>

                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-slate-200 pb-4">
                  <div>
                    <h2 class="text-lg font-extrabold text-slate-900">Attendance Register — Course ${fp.selectedCourse}</h2>
                    <p class="text-xs text-slate-500">Showing key roll numbers from ${cohortDisplayTotal}-student cohort • Committing syncs live to Ashi Golhani's Student Portal.</p>
                  </div>
                  <div class="flex flex-wrap items-center gap-2">
                    <span class="px-3 py-1.5 rounded-xl bg-slate-100 text-slate-800 text-xs font-bold">
                      Marked Present: ${presentSample} / ${filteredRoster.length} Sampled (Cohort: ${cohortDisplayTotal})
                    </span>
                    <button onclick="markFilteredFacultyAttendance(true)" class="px-3 py-1.5 rounded-xl bg-emerald-100 hover:bg-emerald-200 text-emerald-900 text-xs font-bold cursor-pointer">
                      All Present
                    </button>
                    <button onclick="saveFacultyAttendanceToERP()" class="px-4 py-1.5 rounded-xl bg-mits-navy hover:bg-indigo-900 text-white text-xs font-bold shadow cursor-pointer">
                      <i class="fa-solid fa-cloud-arrow-up mr-1"></i> Commit to SAMS ERP
                    </button>
                  </div>
                </div>

                <div class="overflow-x-auto">
                  <table class="w-full text-left text-xs">
                    <thead class="bg-slate-50 text-slate-500 uppercase border-b border-slate-200">
                      <tr>
                        <th class="py-3 px-4">Enrollment No.</th>
                        <th class="py-3 px-4">Student Name</th>
                        <th class="py-3 px-4">Lab Group</th>
                        <th class="py-3 px-4 text-right">Status Toggle</th>
                      </tr>
                    </thead>
                    <tbody class="divide-y divide-slate-100">
                      ${filteredRoster.map(stu => `
                        <tr class="${stu.roll === 'BTEC26O1022' ? 'bg-amber-50/70' : ''}">
                          <td class="py-3 px-4 font-mono font-bold text-slate-800">${stu.roll}</td>
                          <td class="py-3 px-4 font-semibold text-slate-900">
                            ${stu.name} ${stu.roll === 'BTEC26O1022' ? '<span class="ml-1.5 px-2 py-0.5 rounded bg-amber-300 text-mits-navy text-[10px] font-extrabold">Synced Student</span>' : ''}
                          </td>
                          <td class="py-3 px-4 text-slate-500">${stu.batch}</td>
                          <td class="py-3 px-4 text-right">
                            <button onclick="toggleFacultyStudentAttendance('${stu.roll}')"
                              class="px-4 py-1.5 rounded-xl text-xs font-extrabold transition cursor-pointer ${stu.present ? 'bg-emerald-600 text-white' : 'bg-rose-600 text-white'}">
                              ${stu.present ? 'PRESENT (P)' : 'ABSENT (A)'}
                            </button>
                          </td>
                        </tr>
                      `).join('')}
                    </tbody>
                  </table>
                </div>
              </div>
            ` : ''}

            ${state.facultyTab === 'quiz-builder' ? `
              <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <form onsubmit="addDraftQuizQuestion(event)" class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3.5">
                  <h3 class="text-base font-extrabold text-slate-900">Step 1: Add Quiz Question</h3>
                  <div class="grid grid-cols-2 gap-2.5">
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Question Type</label>
                      <select id="fq-type" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-semibold bg-white">
                        <option value="mcq">Single Choice MCQ</option>
                        <option value="integer">Integer / Numerical</option>
                      </select>
                    </div>
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Points</label>
                      <input id="fq-points" type="number" min="1" max="10" value="2" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-mono" />
                    </div>
                  </div>
                  <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">Question Prompt</label>
                    <textarea id="fq-text" rows="2" required placeholder="Type question text..." class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs"></textarea>
                  </div>
                  <div class="grid grid-cols-2 gap-2">
                    <input id="fq-opt1" type="text" placeholder="Option 1 (for MCQ)" class="px-2.5 py-1.5 rounded-lg border border-slate-300 text-xs" />
                    <input id="fq-opt2" type="text" placeholder="Option 2 (for MCQ)" class="px-2.5 py-1.5 rounded-lg border border-slate-300 text-xs" />
                    <input id="fq-opt3" type="text" placeholder="Option 3 (for MCQ)" class="px-2.5 py-1.5 rounded-lg border border-slate-300 text-xs" />
                    <input id="fq-opt4" type="text" placeholder="Option 4 (for MCQ)" class="px-2.5 py-1.5 rounded-lg border border-slate-300 text-xs" />
                  </div>
                  <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">Correct Option # (1–4) or Integer Answer</label>
                    <input id="fq-correct" type="number" value="1" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-mono" />
                  </div>
                  <button type="submit" class="w-full py-2.5 rounded-xl bg-indigo-900 hover:bg-indigo-800 text-white text-xs font-bold cursor-pointer">
                    <i class="fa-solid fa-plus mr-1"></i> Add Question to Draft
                  </button>
                </form>

                <form onsubmit="publishFacultyQuizToStudents(event)" class="lg:col-span-2 bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-4">
                  <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-200 pb-3">
                    <div>
                      <h3 class="text-base font-extrabold text-slate-900">Step 2: Configure & Push Quiz to Student Portal</h3>
                      <p class="text-xs text-slate-500">Published quizzes appear immediately in the Student Portal with DND & 2-Strike Anti-Cheat enabled.</p>
                    </div>
                    <span class="px-2.5 py-1 rounded-full bg-amber-100 text-amber-900 text-xs font-bold">${state.facultyQuizDraftQuestions.length} Questions Ready</span>
                  </div>

                  <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Course</label>
                      <select id="fq-course" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-semibold bg-white">
                        ${state.courses.filter(c => c.type === 'Theory').map(c => `<option value="${c.code}" ${c.code === '33261102' ? 'selected' : ''}>${c.code} – ${c.shortName}</option>`).join('')}
                      </select>
                    </div>
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Quiz Title</label>
                      <input id="fq-title" type="text" required value="In-Class Quiz #5: Pointers & Dynamic Memory in C"
                        class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs" />
                    </div>
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Timer (Minutes)</label>
                      <input id="fq-duration" type="number" min="2" max="60" value="10"
                        class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-mono" />
                    </div>
                  </div>

                  <div class="space-y-2 max-h-56 overflow-y-auto pr-1">
                    ${state.facultyQuizDraftQuestions.map((dq, i) => `
                      <div class="p-3 rounded-xl bg-slate-50 border border-slate-200 flex items-start justify-between gap-3 text-xs">
                        <div>
                          <span class="font-bold text-indigo-800">Q${i + 1} (${dq.type.toUpperCase()} • ${dq.points} pts):</span>
                          <span class="text-slate-800 ml-1">${dq.text}</span>
                        </div>
                        <button type="button" onclick="state.facultyQuizDraftQuestions.splice(${i},1); render();" class="text-rose-600 hover:text-rose-800 font-bold cursor-pointer">
                          <i class="fa-solid fa-trash"></i>
                        </button>
                      </div>
                    `).join('')}
                  </div>

                  <button type="submit" class="w-full py-3 rounded-xl bg-emerald-700 hover:bg-emerald-600 text-white text-xs font-extrabold shadow cursor-pointer">
                    <i class="fa-solid fa-paper-plane mr-1.5"></i> Launch & Publish Live Quiz to Student Login
                  </button>
                </form>
              </div>
            ` : ''}

            ${state.facultyTab === 'marks' ? `
              <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-5">
                <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 border-b border-slate-200 pb-4">
                  <div>
                    <h2 class="text-lg font-extrabold text-slate-900">4-Tier Assessment Marks Entry (Student: Ashi Golhani – BTEC26O1022)</h2>
                    <p class="text-xs text-slate-500">Select an assessment tier below to update marks. Changes immediately sync to the Student Portal's Scorebook.</p>
                  </div>
                  <div class="flex flex-wrap gap-1.5 bg-slate-100 p-1 rounded-xl">
                    <button onclick="state.facultyMarksTier='classQuiz'; render();" class="px-3 py-1.5 rounded-lg text-xs font-bold cursor-pointer ${state.facultyMarksTier === 'classQuiz' ? 'bg-mits-navy text-white' : 'text-slate-600'}">1. Class Quizzes</button>
                    <button onclick="state.facultyMarksTier='centralQuiz'; render();" class="px-3 py-1.5 rounded-lg text-xs font-bold cursor-pointer ${state.facultyMarksTier === 'centralQuiz' ? 'bg-mits-navy text-white' : 'text-slate-600'}">2. Central Level Quiz</button>
                    <button onclick="state.facultyMarksTier='minor'; render();" class="px-3 py-1.5 rounded-lg text-xs font-bold cursor-pointer ${state.facultyMarksTier === 'minor' ? 'bg-mits-navy text-white' : 'text-slate-600'}">3. Minor Exams</button>
                    <button onclick="state.facultyMarksTier='major'; render();" class="px-3 py-1.5 rounded-lg text-xs font-bold cursor-pointer ${state.facultyMarksTier === 'major' ? 'bg-mits-navy text-white' : 'text-slate-600'}">4. Major Exams</button>
                  </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                  ${state.courses.map(c => {
                    if (state.facultyMarksTier === 'centralQuiz' && c.type === 'Lab') return '';
                    let curVal = '';
                    let maxLabel = '';
                    if (state.facultyMarksTier === 'classQuiz') { curVal = c.classQuizMarks; maxLabel = `/ ${c.classQuizMax} Marks`; }
                    else if (state.facultyMarksTier === 'centralQuiz') { curVal = c.centralQuiz; maxLabel = `/ 20 Marks (CBT)`; }
                    else if (state.facultyMarksTier === 'minor') { curVal = c.minorMarks ?? ''; maxLabel = `/ 30 Marks`; }
                    else if (state.facultyMarksTier === 'major') { curVal = c.majorMarks ?? ''; maxLabel = `/ 50 Marks`; }

                    return `
                      <div class="p-4 rounded-2xl border border-slate-200 bg-slate-50 flex items-center justify-between gap-4">
                        <div>
                          <div class="font-mono text-xs font-bold text-indigo-800">${c.code} (${c.type})</div>
                          <div class="font-bold text-slate-900 text-sm">${c.name}</div>
                        </div>
                        <div class="flex items-center gap-2 shrink-0">
                          <input type="number" step="0.5" min="0" max="50" value="${curVal}" placeholder="Pending"
                            onchange="updateAshiTierMark('${c.code}', '${state.facultyMarksTier}', this.value)"
                            class="w-24 px-3 py-2 rounded-xl border border-slate-300 bg-white font-mono text-xs font-bold text-right" />
                          <span class="text-xs text-slate-500 font-medium">${maxLabel}</span>
                        </div>
                      </div>
                    `;
                  }).join('')}
                </div>

                <div class="flex justify-end pt-2">
                  <button onclick="render(); showToast('All 4-tier assessment marks synced to Student Scorebook!', 'success');"
                    class="px-5 py-2.5 rounded-xl bg-emerald-700 hover:bg-emerald-600 text-white text-xs font-bold cursor-pointer">
                    <i class="fa-solid fa-check-double mr-1.5"></i> Sync Marks to Student Portal
                  </button>
                </div>
              </div>
            ` : ''}

            ${state.facultyTab === 'materials' ? `
              <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <form onsubmit="uploadFacultyMaterial(event)" class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-4">
                  <h3 class="text-base font-extrabold text-slate-900">Upload Unit Study Resource</h3>
                  <p class="text-xs text-slate-500">Uploaded files appear immediately inside the Student Portal's 5-Unit Syllabus Repository.</p>
                  <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">Course</label>
                    <select id="fac-mat-course" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-semibold">
                      ${state.courses.map(c => `<option value="${c.code}" ${c.code === '33261102' ? 'selected' : ''}>${c.code} – ${c.shortName}</option>`).join('')}
                    </select>
                  </div>
                  <div class="grid grid-cols-2 gap-3">
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Syllabus Unit</label>
                      <select id="fac-mat-unit" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-semibold">
                        <option value="1">Unit 1</option>
                        <option value="2">Unit 2</option>
                        <option value="3">Unit 3</option>
                        <option value="4">Unit 4</option>
                        <option value="5">Unit 5</option>
                      </select>
                    </div>
                    <div>
                      <label class="block text-xs font-bold text-slate-700 mb-1">Resource Type</label>
                      <select id="fac-mat-type" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-semibold">
                        <option value="PDF">Lecture PDF</option>
                        <option value="Notes">Handwritten Notes</option>
                        <option value="Question Bank">Question Bank</option>
                      </select>
                    </div>
                  </div>
                  <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">Document Title</label>
                    <input id="fac-mat-title" type="text" required placeholder="e.g. Unit 4 Dynamic Memory Allocation in C"
                      class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs" />
                  </div>
                  <button type="submit" class="w-full py-2.5 rounded-xl bg-mits-navy hover:bg-indigo-900 text-white text-xs font-bold cursor-pointer">
                    <i class="fa-solid fa-upload mr-1.5"></i> Publish Resource to Student Portal
                  </button>
                </form>

                <div class="lg:col-span-2 bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3">
                  <h3 class="text-base font-extrabold text-slate-900">Recently Published Resources (33261102 – Computer Programming)</h3>
                  <div class="space-y-2.5">
                    ${state.courses.find(c => c.code === '33261102').materials.map(m => `
                      <div class="p-3.5 rounded-xl border border-slate-200 bg-slate-50 flex items-center justify-between gap-3">
                        <div>
                          <span class="px-2 py-0.5 rounded bg-amber-100 text-amber-900 text-[10px] font-bold">Unit ${m.unit} • ${m.type}</span>
                          <div class="font-bold text-slate-900 text-xs mt-1">${m.title}</div>
                          <div class="text-[11px] text-slate-400">Uploaded ${m.date} • ${m.size}</div>
                        </div>
                        <span class="text-xs font-bold text-emerald-700"><i class="fa-solid fa-check-double mr-1"></i>Live</span>
                      </div>
                    `).join('')}
                  </div>
                </div>
              </div>
            ` : ''}
          </main>
        </div>
      `;
    }

    function publishNewClubEvent(e) {
      e.preventDefault();
      const title = document.getElementById('club-evt-title').value.trim();
      const venue = document.getElementById('club-evt-venue').value;
      const dateStr = document.getElementById('club-evt-date').value.trim() || "Dec 06, 2026 • 03:00 PM";
      const solo = document.getElementById('club-evt-solo').value.trim() || "Free";
      const duo = document.getElementById('club-evt-duo').value.trim() || "₹100";
      const squad = document.getElementById('club-evt-squad').value.trim() || "₹200";
      const perks = document.getElementById('club-evt-perks').value.trim() || "E-Certificates + Merit Goodies";
      const desc = document.getElementById('club-evt-desc').value.trim();

      if (!title || !desc) return;

      const newEvt = {
        id: `EVT-${Math.floor(200 + Math.random() * 700)}`,
        title,
        club: state.clubProfile.clubName,
        category: "Tech & Societies",
        date: dateStr,
        venue,
        fees: { solo, duo, squad },
        perks,
        desc,
        registrations: 1,
        registered: false
      };

      state.events.unshift(newEvt);
      state.announcements.unshift(`🎉 NEW EVENT PUBLISHED: "${title}" by ${state.clubProfile.clubName} at ${venue} (${dateStr})!`);
      e.target.reset();
      render();
      showToast(`"${title}" published! It is now live in the Student Portal's Events Feed.`, "success");
    }

    function broadcastClubNotice(e) {
      e.preventDefault();
      const msg = document.getElementById('club-broadcast-msg').value.trim();
      if (!msg) return;
      state.announcements.unshift(`📣 [${state.clubProfile.clubName}]: ${msg}`);
      document.getElementById('club-broadcast-msg').value = "";
      render();
      showToast("Announcement pushed to SAMS Campus Live Ticker!", "success");
    }

    function deleteClubEvent(id) {
      state.events = state.events.filter(e => e.id !== id);
      render();
      showToast("Event removed from campus feed.", "info");
    }

    function renderClubPortal() {
      const cp = state.clubProfile;
      return `
        <div class="min-h-screen bg-slate-100 flex flex-col">
          <header class="bg-gradient-to-r from-emerald-950 via-teal-900 to-mits-navy text-white px-4 sm:px-8 py-4 shadow-lg">
            <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-4">
              <div class="flex items-center gap-3.5">
                ${getMitsCrestSVG("w-11 h-11")}
                <div>
                  <div class="flex items-center gap-2">
                    <span class="px-2.5 py-0.5 rounded bg-emerald-400 text-emerald-950 text-[10px] font-extrabold uppercase">${cp.role}</span>
                    <span class="text-xs text-emerald-200 font-mono">Supervisor: ${cp.facultySupervisor}</span>
                  </div>
                  <h1 class="text-lg font-extrabold mt-0.5">${cp.clubName}</h1>
                  <p class="text-xs text-slate-300">Logged in as ${cp.bearerName} (${cp.email})</p>
                </div>
              </div>

              <button onclick="logoutToLanding()" class="px-4 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white text-xs font-bold cursor-pointer">
                <i class="fa-solid fa-right-from-bracket mr-1"></i> Switch Portal
              </button>
            </div>
          </header>

          <main class="flex-1 max-w-7xl w-full mx-auto p-4 sm:p-6 grid grid-cols-1 lg:grid-cols-3 gap-6">
            <div class="space-y-6">
              <form onsubmit="publishNewClubEvent(event)" class="bg-white rounded-2xl border border-slate-200 p-6 shadow-xs space-y-3.5">
                <div class="flex items-center justify-between">
                  <h2 class="text-base font-extrabold text-slate-900">Publish New Event / Workshop</h2>
                  <span class="px-2 py-0.5 rounded bg-emerald-100 text-emerald-800 text-[10px] font-bold">Live Sync</span>
                </div>
                <div>
                  <label class="block text-xs font-bold text-slate-700 mb-1">Event / Workshop Title</label>
                  <input id="club-evt-title" type="text" required placeholder="e.g. Edge AI & ESP32 Bootcamp 2026"
                    class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs" />
                </div>
                <div class="grid grid-cols-2 gap-2.5">
                  <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">Venue (SH-1 to SH-5)</label>
                    <select id="club-evt-venue" class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-semibold bg-white">
                      <option value="SH-1 (Seminar Hall 1)">SH-1 (Seminar Hall 1)</option>
                      <option value="SH-2 (Seminar Hall 2)">SH-2 (Seminar Hall 2)</option>
                      <option value="SH-3 (Seminar Hall 3)" selected>SH-3 (Seminar Hall 3)</option>
                      <option value="SH-4 (Seminar Hall 4)">SH-4 (Seminar Hall 4)</option>
                      <option value="SH-5 (Seminar Hall 5)">SH-5 (Seminar Hall 5)</option>
                      <option value="CIOT Block Lab M005">CIOT Block Lab M005</option>
                    </select>
                  </div>
                  <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">Date & Time</label>
                    <input id="club-evt-date" type="text" value="Dec 06, 2026 • 03:00 PM"
                      class="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs" />
                  </div>
                </div>

                <div class="grid grid-cols-3 gap-2">
                  <div>
                    <label class="block text-[11px] font-bold text-slate-600 mb-1">Solo Fee</label>
                    <input id="club-evt-solo" type="text" value="Free" class="w-full px-2.5 py-1.5 rounded-lg border border-slate-300 text-xs font-mono" />
                  </div>
                  <div>
                    <label class="block text-[11px] font-bold text-slate-600 mb-1">Duo Fee</label>
                    <input id="club-evt-duo" type="text" value="₹99" class="w-full px-2.5 py-1.5 rounded-lg border border-slate-300 text-xs font-mono" />
                  </div>
                  <div>
                    <label class="block text-[11px] font-bold text-