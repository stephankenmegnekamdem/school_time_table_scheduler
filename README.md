# school_time_table_scheduler

A console program written in Java that builds a weekly timetable for six classes. It asks for the school's classes, subjects, required hours, teachers, teacher absences, class teacher assignments, and teaching eligibility. It then places the fixed lessons (religion, social hour, and arts), generates the remaining lessons with a randomized greedy search, and validates the result against every rule.

## Table of contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Running the program](#running-the-program)
- [Using the program](#using-the-program)
- [How the timetable works](#how-the-timetable-works)
- [Fixed lessons](#fixed-lessons)
- [Scheduling rules](#scheduling-rules)
- [Output](#output)
- [Project structure](#project-structure)
- [How the program works internally](#how-the-program-works-internally)
- [Troubleshooting](#troubleshooting)
- [Known limitations](#known-limitations)
- [Status](#status)
- [License](#license)

## Features

- Interactive console input for classes, subjects, hours, and teachers
- Class teacher assignment, with one class teacher per class
- Teacher absences by time slot
- Teaching eligibility per teacher, class, and subject
- Automatic placement of fixed lessons: Rel and Socialhour for every class, Arts for classes 2 to 6
- Randomized search with up to 1000 attempts
- Validation of fixed lessons, teacher conflicts, absences, daily subject limits, exact subject hours, and eligibility
- Printing of the full timetable with fixed lessons marked
- Printing of the best attempt and a score breakdown when no perfect timetable is found

## Requirements

- Java 8 or newer (the code uses lambdas and standard collections only)
- A Java Development Kit (JDK), not just a runtime
- No external libraries
- Optional: IntelliJ IDEA, the IDE the project was written in

To check your installation, run:

```bash
java -version
javac -version
```

Both commands should print a version number.

## Installation

Clone the repository:

```bash
git clone git@github.com:stephankenmegnekamdem/school_time_table_scheduler.git
git clone https://github.com/stephankenmegnekamdem/school_time_table_scheduler.git
cd school-timetable-generator
```


## Running the program

### From IntelliJ IDEA

1. Open the project folder (the one that contains `src`).
2. In the Project panel, right-click `src/Main.java`.
3. Choose **Run 'Main.main()'**.
4. Type your input in the Run console.

### From the command line

From the project folder, compile all sources into an `out` folder and run `Main`:

```bash
javac -d out src/*.java
java -cp out Main
```

On Windows, if `src/*.java` does not expand in your shell, list the files explicitly or use IntelliJ.

## Using the program

The program asks for input in this order.

### 1. Subjects for each class

For classes 1 to 6, the program asks:

1. The number of subjects for the class.
2. For each subject, its name, then the number of hours per week.

Subject names are typed as single words. Use `Math`, not `Mathematics Advanced`, because the program reads input with `next()`, which stops at the first space.

Do not enter `Rel`, `Socialhour`, or `Arts` here. These are placed automatically.

### 2. Teachers

The program asks for the number of teachers. For each teacher it asks, in order:

1. **Name.** Single word, unique across all teachers.
2. **Number of absences**, then the absence slots. Enter each absence as a slot number from 1 to 30. Invalid numbers are rejected and asked again.
3. **Class teacher?** Answer `yes` or `no`. If `yes`, enter the class number (1 to 6). A class that already has a class teacher is rejected.
4. **Number of classes the teacher can teach**, then for each class:
   - the class number (1 to 6),
   - the number of subjects the teacher can teach in that class,
   - for each subject, the subject number from the list shown.

A teacher can be eligible for several subjects in several classes.

### 3. Result

The program prints one of two outcomes:

- `TIMETABLE GENERATION SUCCESSFUL.` followed by the timetable, if every rule is satisfied.
- `NO PERFECT TIMETABLE FOUND.` followed by `BEST AVAILABLE TIMETABLE:`, the timetable, and a `BEST TIMETABLE ANALYSIS` section with the penalty counts.

Finally, it lists the fixed lessons and the number of classes and teachers.

### Example session (abbreviated)
