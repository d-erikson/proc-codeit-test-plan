# Test plan for PROC CODEIT enhancement

## 1. Introduction

### 1.1 Purpose

The purpose of this test plan is to define the overall strategy for testing the enhancement to the `DO statement` in PROC CODEIT. This plan provides:
- A structured framework for designing and executing the tests
- Coverage of functional, boundary, error-handling, and performance scenarios
- Criteria for evaluating the readiness of the enhancement

### 1.2 Project Overview

PROC CODEIT currently supports the following syntax for the `DO statement`:

```
DO index = start TO stop;
	<statement-list>
END;
```

The index variable always increments by 1.

A customer requested an enhancement to introduce an optional `BY increment` clause, allowing the user to specify a custom step value:

```
DO index = start TO stop BY increment;
	<statement-list>
END;
```

This enhancement provides greater flexibility in the looping for PROC CODEIT.

## 2. Scope

### 2.1 In scope

- Numeric ranges with positive/negative/fractional increments
- Termination at boundaries
- Error handling and diagnostics 
- Behavior when start > stop with positive increment (0 iterations), and vice versa
- Nested loops
- Large iteration counts 

### 2.2 Out of scope 

- Non-DO statements: IF, WHILE, and other statements are not affected and are covered by existing regression testing.
- Cross-product compatibility: The scope is limited to validating syntax and execution inside PROC CODEIT.
- Integration with external data sources: The `BY increment` feature is internal to PROC CODEIT.
- Security testing: This enhancement does not introduce new user inputs or external integrations.

## 3. Test Approach

- Functional Testing: Verify core functionality of `BY increment`.
- Boundary Testing: Test edges of input ranges.
- Error Handling: Ensure meaningful errors for invalid syntax.
- Performance Testing: Verify loop efficiency and stability under extreme circumstances.
- Regression Testing: Verify no bugs introduced into existing `DO statement` functionality.

## 4. Test Scenarios

### 4.1 Functional Tests

| ID    | Type       | Scenario                               | Input                            | Expected Result                                                      |
|-------|------------|----------------------------------------|----------------------------------|----------------------------------------------------------------------|
| TC-01 | Functional | Positive integer increment             | `1 TO 10 BY 2`                   | Index = 1, 3, 5, 7, 9                                                |
| TC-02 | Functional | Negative integer increment             | `10 TO 1 BY -2`                  | Index = 10, 8, 6, 4, 2                                               |
| TC-03 | Functional | Positive fractional increment          | `1 TO 3 BY 0.5`                  | Index = 1, 1.5, 2, 2.5, 3                                            |
| TC-04 | Functional | Negative fractional increment          | `3 TO 1 BY -0.5`                 | Index = 3, 2.5, 2, 1.5, 1                                            |
| TC-05 | Functional | Mixed types for start/stop/by          | `1.5 TO 4 BY 0.5`                | Index = 1.5, 2, 2.5, 3, 3.5, 4                                       |
| TC-06 | Functional | Negative range (ascending fractional)  | `-3 TO -1 BY 0.5`                | Index = -3, -2.5, -2, -1.5, -1                                       |
| TC-07 | Functional | Negative range (ascending integer)     | `-10 TO -1 BY 2`                 | Index = -10, -8, -6, -4, -2                                          |
| TC-08 | Functional | Negative range (descending fractional) | `-1 TO -3 BY -0.5`               | Index = -1, -1.5, -2, -2.5, -3                                       |
| TC-09 | Functional | Negative range (descending integer)    | `-2 TO -10 BY -2`                | Index = -2, -4, -6, -8, -10                                          |
| TC-10 | Functional | Nested positive increments             | `i: 1 TO 5 BY 2; j: 1 TO 7 BY 3` | Index = (1,1), (1,4), (1,7), (3,1), (3,4), (3,7), (5,1), (5,4), (5,7)|
| TC-11 | Functional | Start/stop/by values from variables    | `start=1; stop=10; step=2; start TO stop BY step; …` | Index = 1, 3, 5, 7, 9                            |

### 4.2 Boundary Tests

| ID    | Type     | Scenario                                                      | Input                            | Expected Result                                 |
|-------|----------|---------------------------------------------------------------|----------------------------------|-------------------------------------------------|
| TC-12 | Boundary | Stops exactly at boundary end (positive increment)            | `1 TO 10 BY 3`                   | Index = 1, 4, 7, 10                             |
| TC-13 | Boundary | Overshoots boundary end (positive increment)                  | `1 TO 10 BY 4`                   | Index = 1, 5, 9 (does not exceed stop)          |
| TC-14 | Boundary | Stops exactly at boundary end (negative increment)            | `10 TO 1 BY -3`                  | Index = 10, 7, 4, 1                             |
| TC-15 | Boundary | Overshoots boundary end (negative increment)                  | `10 TO 1 BY -4`                  | Index = 10, 6, 2 (does not go below stop)       |
| TC-16 | Boundary | Stops exactly at boundary end (positive fractional increment) | `1 TO 3 BY 0.5`                  | Index = 1, 1.5, 2, 2.5, 3                       |
| TC-17 | Boundary | Overshoots boundary end (positive fractional increment)       | `1 TO 3 BY 0.6`                  | Index = 1, 1.6, 2.2, 2.8 (does not exceed stop) |
| TC-18 | Boundary | Stops exactly at boundary end (negative fractional increment) | `3 TO 1 BY -0.5`                 | Index = 3, 2.5, 2, 1.5, 1                       |
| TC-19 | Boundary | Overshoots boundary end (negative fractional increment)       | `3 TO 1 BY -0.6`                | Index = 3, 2.4, 1.8, 1.2 (does not go below stop)|
| TC-20 | Boundary | Start and stop equal                                          | `5 TO 5 BY 1`                    | Executes once (index = 5)                       |
| TC-21 | Boundary | BY larger than range (positive increment)                     | `1 TO 5 BY 10`                   | Executes once (index = 1)                       |
| TC-22 | Boundary | BY larger than range (negative increment)                     | `5 TO 1 BY -10`                  | Executes once (index = 5)                       |
| TC-23 | Boundary | Overshoots boundary end (nested loops positive increment)     | `i: 1 TO 6 BY 4; j: 1 TO 5 BY 3` | Index = (1,1), (1,4), (5,1), (5,4)              |
| TC-24 | Boundary | Floating-point termination    | `0 TO 1 BY 0.1`      | Index = 0.0, 0.1, 0.2, ..., 0.9, 1.0 (terminates correctly despite floating-point precision)|

### 4.3 Error Handling/Negative Tests
| ID    | Type     | Scenario                               | Input                                         | Expected Result                                           |
|-------|----------|----------------------------------------|-----------------------------------------------|-----------------------------------------------------------|
| TC-25 | Negative | Zero increment                         | `1 TO 10 BY 0`                                | Error (ensure no infinite loop)                           |
| TC-26 | Negative | Non-numeric increment                  | `1 TO 10 BY 'A'`                              | Syntax / type error                                       |
| TC-27 | Negative | Mismatched ranges (negative increment) | `1 TO 10 BY -1`                               | No iterations / no error                                  |
| TC-28 | Negative | Mismatched ranges (positive increment) | `10 TO 1 BY 1`                                | No iterations / no error                                  |
| TC-29 | Negative | Invalid types for `BY` keyword         | `0 TO 3 BY 'abc'`                             | Syntax / type error                                       |
| TC-30 | Negative | Invalid types for STOP/START           | `'A' TO 'Z' BY 1;`                            | Syntax / type error                                       |
| TC-31 | Negative | Missing `BY` keyword                   | `1 TO 5 2;`                                   | Syntax error                                              |
| TC-32 | Negative | `BY` keyword without a value           | `1 TO 5 BY;`                                  | Syntax error                                              |
| TC-33 | Negative | Missing `END`                          | `DO i = 1 TO 3 BY 1; PRINT i;`                | Syntax error - references missing `END`                   |
| TC-34 | Negative | Nested loop with unbalanced END     | `DO i = 1 TO 3 BY 2; DO j = 1 TO 5 BY 3; END;`   | Syntax error - references missing outer `END`             |
| TC-35 | Negative | Nested loop conflict (same index name) | `i: 1 TO 3 BY 2; i: 1 TO 5 BY 3;` | Implementation-dependent - Verify if inner loop reuses/overwrites `i` |
| TC-36 | Negative | Index modification            | `DO i = 1 TO 5 BY 2; IF i=2 THEN i=100; END;` | Implementation-dependent - Verify if index modification is ignored |

Note TC-35 and TC-36 are discovery tests. Outcomes are implementation-dependent. Testers should document observed behavior for spec clarification.

### 4.4 Performance Tests
| ID    | Type        | Scenario             | Input                  | Expected Result                                                                     |
|-------|-------------|----------------------|------------------------|-------------------------------------------------------------------------------------|
| TC-37 | Performance | High-iteration range | `1 TO 1000000 BY 2`    | Completes 500000 times - executes without crash, hang, or unacceptable slowdown     |
| TC-38 | Performance | Wide-step increment  | `1 TO 5 BY 1000000`    | Executes once (index = 1) - executes without crash, hang, or unacceptable slowdown  |

### 4.5 Regression Tests
| ID     | Type        | Scenario                        | Input                  | Expected Result                                                                     |
|--------|-------------|---------------------------------|------------------------|-------------------------------------------------------------------------------------|
| TC-39  | Regression  | Omission of increment           | `1 TO 3`               | Defaults to increment = 1 - behavior already validated by existing regression suite |

**Priorities:**  
- P1 = Functional and Boundary tests  
- P2 = Negative and Regression tests  
- P3 = Performance and Discovery tests  

## 5. Assumptions / Risks

### 5.1 Assumptions

- `BY increment` accepts fractional values. Loop termination allows for minor floating-point rounding differences when comparing to the stop value.
- Default increment: If `BY increment` is omitted, the increment = 1.
- Termination rules:
  - For positive increments: loop runs while index <= stop.
  - For negative increments: loop runs while index >= stop.
- If the index would overshoot the stop, the loop does not execute that iteration. After termination, the index retains the last executed value.
- Zero increment is invalid and will throw an error.
- Start/stop/increment are numeric values (integers, floating-point literals, scientific notation). Non-numeric increments will throw an error.
- Index visibility: index is readable inside the loop. Behavior when modifying the index is implementation-dependent.
- Nested DO statements are allowed.
- If start == stop, loop runs exactly once.
- Start/stop/increment may be specified as literals or variables. Behavior is consistent for both cases.

### 5.2 Risks

- Increment logic may not handle floating-point precision correctly.
- Index modification inside body may cause unexpected skips or repeats.
- Very large start/stop ranges or very small `BY` increments may surface performance issues. 
- Zero increment could cause an infinite loop.
- Ambiguous errors (like "invalid loop") could slow triage.

## 6. Test Data 

- Representative test data (small/large, positive/negative, edge/error cases) is described within the Test Scenarios in section 4.
- No separate test data files required. Scenario definitions contain needed data.

## 7. Deliverables

- Test Plan Document
- Test Cases 
- Test Results Report (summary of executed tests, pass/fail status, and any defects)

## 8. Example Test Case 

Test Case ID: TC-11
- Objective: Verify that the DO loop correctly handles a positive integer increment value.
- Test Scenario: Loop from a lower value to a higher value with a positive integer increment. Use start/stop/by values that are stored as variables.

```
// Setup
Create an empty list called result_list
Set start = 1
Set stop = 10
Set step = 2

// Execute the loop
PROC CODEIT;
	DO index = start TO stop BY step;
		Add index to result_list
	END;
QUIT;

// Expected result
expected_output = [1, 3, 5, 7, 9]

// Verify
IF result_list equals expected_output
	PRINT "TEST PASSED: Positive integer increment test passed."
ELSE
	PRINT "TEST FAILED: Positive integer increment test failed."
	PRINT "Expected: " + expected_output
	PRINT "Actual: " + result_list
ENDIF
```

## 9. Exit Criteria

- All planned test cases executed.
- 100% pass rate on P1 test cases. 
- No critical severity defects outstanding.

## 10. References

- Customer request for `BY increment` enhancement.
- PROC CODEIT documentation.
- Existing regression test suite for `DO statement` (baseline behavior) 