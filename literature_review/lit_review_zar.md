# Literature Review: Test Error Grouping/Crash Bucketing

## Semantic Crash Bucketing
**Author(s):** Rijnard van Tonder, John Kotheimer, Claire Le Goues 
**Year of Publication:** 2018  
**Link/DOI:** https://dl.acm.org/doi/pdf/10.1145/3238147.3238200

---

### Intro/Summary
Semantic Crash Bucketing (SCB) is an approach to automatically identify unique bugs by modifying a program's semantics. It groups crashing inputs based on program transformations that nullify the crashes. The approach uses bug-fixing patch templates and rule-based patch applications to approximate correct fixes. SCB ensures that crashes are grouped according to program changes that nullify those inputs, providing a way to categorize bugs semantically.

---

### Approach
Here is the SCB approach:
![img.png](imgs/SemanticCrashBucketing.png)
The developer's fix provides the best assurance of correctly fixing a bug, which is the ground truth T for SCB. But how do they obtain the approximate fix?
#### For Null Dereferences:
They inserted a simple **exit(101)** instruction when the variable is **null**. 
What about finding the variable and its location?
The general procedure for finding such crash-inducing variables works as follows:
1. Attach **GDB** to the program and run it on the crashing input.
2. Extract the source line and code reported at the crash.
3. Parse the code for pointer dereference syntax (e.g., `p->q`).
4. Working backwards, extract program variables that are dereferenced (e.g., extract p from p->q). Test, using GDB, whether the variable is null in the debugger environment.
5. If the variable is null, return the variable and associated line number. If not, move backwards a basic block and continue from (3).
If the procedure succeeds, we substitute the template program variable and insert the candidate patch just before the null dereference.
##### For Buffer Overflows:
Buffer overflows are typically fixed by performing array bounds checking on memory accesses. Thus they focus on array length. they rewrite existing calls and restrict the length of data copied to a default concrete value of 1. Restricting data to only one byte approximates a conservative angelic value that is likely to lead to non-crashing program termination.
They modified the possible problematic call (**memcpy** for example) by giving an angelic length to the array. 
How to find the problematic library calls?
The steps are as follows:
1. Use ***ltrace*** to obtain a trace of library calls from the crashing program run.
2. Working backwards, resolve the source location of library calls in the trace for which we have fixing templates.
3. Apply the template at the location and rerun the program on the original crashing input.
4. If the program no longer crashes, emit the approximate fixing patch TD. Else continue from step.

#### Tools
- **GDB:** Debugger used to find crash-inducing variables in null dereferences.
- **ltrace:** Tool to trace library calls for buffer overflow analysis.

### Dataset
- **Source:** CVE database. https://cve.mitre.org/
- **Bug Types:** Buffer overflows and null dereferences.
- **Experimental Setup:** Controlled experiments with real bugs where the ground truth fixes are known.

---

### Tags
- Crash Bucketing, Fuzzing, Bug Triage, Program Transformation, Automated Bug Fixing

---

### Notes
- SCB relies heavily on approximate fixes derived from rule-based templates and debugging tools.
- The developer’s actual fix is used as the ground truth for validating the approach.

