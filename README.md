# Contribution-README
# Contribution [1]: Add option to write logs to a file #2183

**Contribution Number:** [1]  
**Student:** Paul Tsekpo  
**Issue:** https://github.com/dosbox-staging/dosbox-staging/issues/2183  
**Status:** [Phase I] [In Progress]

---

## Why I Chose This Issue

I chose this Issue because I wanted to gain some experience with the loguru logging module

---

## Understanding the Issue

### Problem Description

The issue is that there is no option to log to files once teh excutable starts up. The only option available to the user was to log to the console. 
In other words the loguru plugin that can help with logging to a file wasnt implemented
### Expected Behavior

User would have options to choose between.
Options to log to file, console, both or none 

### Current Behavior
The only option available to teh user was to log to the console


### Affected Components

The main.cpp, dosbox.cpp where the main components affected

---

## Reproduction Process

### Environment Setup
The environment setup of this stage was to clone the repo on my local(vs code). From there, proceeded to intall the vckpg dependencies and set the environment variable path for it
Now from there build the code to make sure it was able to build properly.

[Notes on setting up your local development environment - challenges you faced, how you solved them]


### Steps to Reproduce

1. In vs-code click on build(this builds the program and creates an executable in ./build/resources/resources-windows2022/dosbox.exe)
2. Run the exe file
3. Program is up and running and logs are sent to the console(before rolling logs/log to file is implemented)

### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork](https://github.com/dosbox-staging/dosbox-staging/compare/main...blackheart-5:dosbox-staging:main)]
- **Screenshots/logs:** [If applicable]
- **My findings:** [during production logs are logged on the console(no option to log to a file)]

---

## Solution Approach

### Analysis

It doesnt have an option for users to log into file. Config settings lacks that feature so by default logs are sent to console.

### Proposed Solution

In dosbox.cpp add the config settings that gives user the option of where logs should be sent.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Config settings lacks that feature so by default logs are sent to console.]

**Match:** [There are other config settings that match what we are expected to do. An example is the machine config setting]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. From the section add a string name of the config settings and
3. later add the path to that
4. create functions to find the next index of logs
5. create function to delete log file/oldest
6. create function iterator that maps fils in the dir

**Implement:** [https://github.com/dosbox-staging/dosbox-staging/compare/main...blackheart-5:dosbox-staging:main]

**Review:** [yes folows contributing.md and convention commit messages]

**Evaluate:** [Check that logs folder containing log files is created in the config dir(also config file must also be editted by user to get the logs where user wants)]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
