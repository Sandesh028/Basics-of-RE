# Basics-of-RE
---


# 🧠 Reverse Engineering with GDB: Vulnerable vs Benign Programs

Welcome to this hands-on exercise on **Reverse Engineering** and **Memory Analysis** using `GDB`!  
In this session, you will analyze two compiled binaries:

- 🧩 `vuln_selftrigger` — intentionally vulnerable program  
- 🧩 `benign` — safe version using secure coding practices  

You will use **GDB (GNU Debugger)** to inspect how each program behaves in memory and learn to recognize signs of stack corruption.

---

## 🧰 Prerequisites

### Install Required Tools
```bash
sudo apt update
sudo apt install -y gdb pwndbg python3 gcc
````

> 💡 *Optional:* Use `pwndbg` for an enhanced GDB interface (automatically loads on some systems).

### Verify Files

Make sure your working directory contains:

```bash
ls
benign  vuln_selftrigger
```

---

## 🧪 Part 1 — Running Both Programs

### 1️⃣ Benign Program

Run it with normal input:

```bash
./benign Alice
```

Expected Output:

```
Hello, Alice!
```

Try longer input:

```bash
./benign $(python3 -c "print('A'*100)")
```

✅ No crash — safe output is truncated.

---

### 2️⃣ Vulnerable Program

Run it in self-trigger mode:

```bash
./vuln_selftrigger --self
```

Expected Output:

```
[self-trigger] calling vuln() with long payload (len=254)
Segmentation fault (core dumped)
```

💥 The program crashes because a long string overflows the stack buffer.

---

## 🧩 Part 2 — Debugging the Crash in GDB

We’ll now open the vulnerable binary inside **GDB** and inspect what happened.

### 1️⃣ Start GDB

```bash
gdb -q ./vuln_selftrigger
```

### 2️⃣ Run the Program

```bash
(gdb) run --self
```

When it crashes, you’ll see:

```
Program received signal SIGSEGV, Segmentation fault.
```

---

## 🔍 Part 3 — Inspecting the Crash

### 1️⃣ View the Backtrace

Shows which functions were called before the crash:

```bash
(gdb) bt full
```

Look for `vuln()` near the top — this is where the overflow occurred.

---

### 2️⃣ View Registers

Registers store temporary CPU data (return addresses, stack pointers, etc.):

```bash
(gdb) info registers
```

Pay attention to:

| Register | Role                        | Observation                                  |
| -------- | --------------------------- | -------------------------------------------- |
| **RIP**  | Current instruction pointer | Usually points into corrupted memory         |
| **RBP**  | Saved frame pointer         | Overwritten with `0x41414141` (ASCII "AAAA") |
| **RSP**  | Stack pointer               | Marks stack top                              |
| **RDI**  | Function argument           | Points to your payload                       |

---

### 3️⃣ Examine the Stack Memory

```bash
(gdb) x/200bx $rsp
```

Scroll to see where `"CRASH"` appears — it marks the overflowed data region.

---

### 4️⃣ Find Your Payload in Memory

```bash
(gdb) find $rsp-0x400, $rsp+0x800, "CRASH"
```

If you see the address printed → payload successfully reached stack memory.

---

## 🧩 Part 4 — Inspecting the Benign Program

Let’s confirm the safe version does **not** crash.

### 1️⃣ Start GDB

```bash
gdb -q ./benign
```

### 2️⃣ Run with Long Input

```bash
(gdb) run $(python3 -c "print('A'*200)")
```

Expected:

```
Hello, AAAAAAAAAAAAAAAAA...
Program exited normally.
```

### 3️⃣ Check Stack (Optional)

```bash
(gdb) info registers
(gdb) x/80bx $rsp
```

✅ The stack remains clean — no overwritten RBP or RIP.

---

## 🧠 Learning Summary

| Concept               | `vuln_selftrigger`  | `benign`            |
| --------------------- | ------------------- | ------------------- |
| Function Used         | `strcpy()` (unsafe) | `snprintf()` (safe) |
| Stack Overflow        | Yes                 | No                  |
| Result                | Crashes             | Normal              |
| Memory Corruption     | ✅ Overwrites RBP    | ❌ Prevented         |
| Protection Mechanisms | Disabled            | Active (safe API)   |

---

## 📚 Key Takeaways

1. **Unsafe C functions (`strcpy`, `gets`)** can overwrite stack memory.
2. **Registers like RBP/RIP** are critical for control flow — overwriting them breaks program execution.
3. **Safe APIs (`snprintf`, `strncpy`)** enforce bounds and prevent corruption.
4. **Reverse engineering tools** (Ghidra + GDB) complement each other:

   * Ghidra → *Static code structure*
   * GDB → *Runtime behavior*

---

## 🧭 Challenge for Students

Try to answer:

1. What value overwrote RBP in `vuln_selftrigger`?
2. Where does `RIP` point when the crash happens?
3. What line in Ghidra corresponds to the crash point?
4. Why does `benign` not show any overwritten registers?

---

## ✅ End of Lab

To exit GDB:

```bash
(gdb) quit
```

