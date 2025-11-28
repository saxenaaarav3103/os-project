
---

````markdown
# 🖥️ CPU Process Scheduling Simulator

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Linux%20%7C%20AWS%20EC2-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Visualization-Matplotlib-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Category-Operating%20Systems-yellow?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Scheduling-FCFS%20%7C%20SJF%20%7C%20Priority%20%7C%20RR-red?style=for-the-badge" />
</p>

---

## ⭐ Project Overview

This project implements a **CPU Process Scheduling Simulator in Python**.  
It includes:

- Classical scheduling algorithms (FCFS, SJF, Priority, Round Robin)  
- Computation of Completion Time (CT), Turnaround Time (TAT), and Waiting Time (WT)  
- Gantt chart visualisation using Matplotlib  
- **Real Linux process integration** using the `ps` command, executed on a **RedHat Linux AWS EC2 instance**

The simulator demonstrates how different scheduling algorithms behave on both a **base test table** and on **live Linux processes**.

---

## ⭐ Algorithm Explanations

### 🔷 1. FCFS — First Come First Serve

| Concept        | Explanation                                   |
|----------------|-----------------------------------------------|
| Selection Rule | Process with earliest Arrival Time (AT) first |
| Preemption     | No                                            |
| CT Formula     | CT = start_time + BT                          |
| Behaviour      | Fair but suffers convoy effect                |

**How It Works:**
- Sort processes by Arrival Time  
- If CPU is idle, jump to next arrival  
- Execute each process fully in arrival order  

---

### 🔷 2. SJF — Shortest Job First (Non-Preemptive)

| Concept        | Explanation                                          |
|----------------|------------------------------------------------------|
| Selection Rule | Among arrived processes, choose shortest BT          |
| Preemption     | No                                                   |
| Advantage      | Produces lowest average WT and TAT                   |
| Caveat         | Long processes may starve                            |
| Behaviour      | Efficient but starvation-prone                       |

**How It Works:**
- At each step, collect all processes with AT ≤ current time  
- Select the one with minimum BT  
- If nothing has arrived yet, fast-forward time to the next arrival  

---

### 🔷 3. Priority Scheduling (Non-Preemptive)

| Concept        | Explanation                                          |
|----------------|------------------------------------------------------|
| Priority Rule  | Lower number = Higher priority                       |
| Preemption     | No                                                   |
| Behaviour      | Favors important tasks but can cause starvation      |

**How It Works:**
- Among all arrived processes, select the highest priority (lowest numeric value)  
- Run it until completion  
- Repeat until all processes are finished  

---

### 🔷 4. Round Robin Scheduling

| Concept        | Explanation                                       |
|----------------|---------------------------------------------------|
| Rule           | Assign CPU time slices using a fixed quantum      |
| Preemption     | Yes                                               |
| Behaviour      | Fair CPU time-sharing between processes           |
| Use Case       | Common in modern multitasking operating systems   |

**How It Works:**
- Maintain a ready queue of processes  
- Each process gets CPU for at most `quantum` time  
- If still unfinished, it re-enters the queue with remaining burst time  
- New arrivals are added into the ready queue dynamically  

---

## ⭐ Formulas Used

### Turnaround Time (TAT)
\[
\text{TAT} = \text{CT} - \text{AT}
\]

### Waiting Time (WT)
\[
\text{WT} = \text{TAT} - \text{BT}
\]

These values are computed in the helper function `TAT_WT()` in the code.

---

## ⭐ Gantt Chart Visualization

Each algorithm generates a **Gantt chart** showing:

- Execution order of processes  
- Start and end times on the time axis  
- Distinct colour for each `Pid`

Generated via:

```python
Gantt_Chart(gantt_data, "Algorithm Name")
````

Where `gantt_data` is a list of tuples of the form:

```python
(Pid, start_time, end_time)
```

---

## ⭐ Linux Integration (Real OS Data)

To integrate real Linux processes, we execute `ps` inside a Linux environment (RedHat EC2 / any Linux):

```bash
ps -eo pid,comm,pri,ni,etimes,stat,pcpu --sort=-pcpu | head -n 20
```

In the Python script, we run a similar command via `subprocess`:

```python
result = subprocess.run(
    ["ps", "-eo", "pid,pri,ni,etimes,comm", "--sort=-pcpu", "--no-headers"],
    capture_output=True,
    text=True
)
```

### Extracted Fields

* `pid`  → Process ID
* `pri`  → Kernel priority
* `ni`   → Nice value
* `etimes` → Elapsed runtime in seconds (used as burst time)
* `comm` → Command name

### Conversion to Scheduler Format

Each row from `ps` is converted into our scheduler process format:

```python
{
    "Pid": str(pid),
    "AT": arrival_order,        # 0, 1, 2, ... (simulated arrival)
    "BT": scaled_elapsed_time,  # derived from etimes
    "Priority": pri
}
```

To keep charts readable, large burst times are scaled:

```python
p["BT"] = max(1, p["BT"] // 50)
```

All four scheduling algorithms (FCFS, SJF, Priority, RR) are then executed on this **Linux-derived dataset** and compared.

---

## ⭐ Repository Structure

```text
.
├── Group Project.ipynb      # Original Jupyter/Colab notebook
├── Group Project.py         # Exported Python script (nbconvert)
├── requirements.txt         # Python dependencies (optional)
└── README.md                # Project documentation
```

> In some environments (e.g. AWS EC2), we run the `.py` script directly after converting from the notebook.

---

## ⭐ How to Run

### 1️⃣ Install Dependencies

Using `pip`:

```bash
pip install matplotlib pandas
```

or with `requirements.txt`:

```bash
pip install -r requirements.txt
```

### 2️⃣ Run the Simulation Locally (Linux / macOS / WSL)

```bash
python3 "Group Project.py"
```

This will:

* Run FCFS, SJF, Priority, and Round Robin on the base test table
* Print CT, TAT, WT and their averages
* Generate Gantt charts for each algorithm
* Fetch real Linux processes via `ps` and rerun all algorithms on that data

### 3️⃣ Run on AWS EC2 (RedHat Linux)

1. Launch a RedHat EC2 instance

2. SSH into the instance

3. Install Python & pip:

   ```bash
   sudo yum install python3 python3-pip -y
   pip3 install matplotlib pandas
   ```

4. Clone your GitHub repository:

   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   cd <your-repo>
   ```

5. (If needed) Convert notebook to script:

   ```bash
   pip3 install jupyter
   jupyter nbconvert --to script "Group Project.ipynb"
   ```

6. Run the script:

   ```bash
   python3 "Group Project.py"
   ```

On headless servers (like EC2), modify the script to use `plt.savefig(...)` instead of `plt.show()` to save Gantt charts as `.png` files.

---

## ⭐ Base Test Table (Custom Dataset)

The simulator is first tested on the following fixed process table:

| PID | AT | BT | Priority |
| --- | -- | -- | -------- |
| P1  | 0  | 6  | 3        |
| P2  | 2  | 4  | 1        |
| P3  | 4  | 5  | 4        |
| P4  | 6  | 3  | 2        |

This table is hard-coded in the script and used to compare average waiting time and turnaround time across all four algorithms.

---

## ⭐ Presentation Checklist

For the project presentation (max 12 slides), this repository supports:

* Introduction & problem statement
* Explanation of FCFS, SJF, Priority, and Round Robin
* Linux command integration:

  * `uname -a`
  * `ps -eo pid,pri,ni,etimes,comm --sort=-pcpu`
* Results and analysis:

  * WT/TAT/CT tables for base test table
  * Linux-derived process table (from `ps`)
  * All Gantt charts (custom + Linux data)
* Team contributions and reflections (who did algorithms, Linux, EC2, PPT, etc.)

---

## ⭐ License

This project is created for **academic use** as part of an Operating Systems course group project and is intended for educational and evaluation purposes only.

```

::contentReference[oaicite:0]{index=0}
```
