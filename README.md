## CBB6340

# Exercise 1: Analyzing Population Data Daily New COVID-19 Cases (Project 9.10)

The following figure compares daily new COVID-19 cases in California,
New York, and Texas.

<img width="1389" height="690" alt="daily_cases" src="https://github.com/user-attachments/assets/cec533f1-68ef-46d7-a197-93018aa8b725" />

### Limitations

Daily case counts were calculated by taking the difference between consecutive
cumulative case totals. This approach may produce negative values when historical
records are corrected. The results may also contain reporting spikes caused by
weekend delays, reporting backlogs, and differences in reporting practices among
states.

### Peak Case Dates Examples

{'state': 'California', 'date': '2022-01-10', 'daily_cases': 227972}
{'state': 'New York', 'date': '2022-01-08', 'daily_cases': 90132}
{'state': 'Texas', 'date': '2022-01-03', 'daily_cases': 164902}

### Compare Dates Examples (CA & NY)
California peak date: 2022-01-10
New York peak date: 2022-01-08
New York reached the peak first.
Days between peaks: 2

### Florida Data Exploration and Anomalies

<img width="1387" height="590" alt="florida daily new cases" src="https://github.com/user-attachments/assets/da18ca8c-e201-476a-b9fb-642167fbe447" />

Florida's daily case data contains several unusual reporting patterns. For
example, the calculated number of new cases on June 4, 2021, is -40,527.
A negative number of actual new infections is not possible, so this observation
likely resulted from a correction to the cumulative total, such as the removal
of duplicate or incorrectly classified cases.

Another anomaly occurs on January 4, 2022, when 193,786 new cases were recorded
after three consecutive days with no reported new cases. This spike may represent
multiple days of cases reported together following the New Year holiday rather
than infections that all occurred on January 4.

The data also contains repeated sequences of zero values followed by large
increases, particularly toward the end of the dataset. A likely explanation is
that Florida changed from daily reporting to less frequent batch reporting.
Therefore, the calculated daily case values represent the dates cases were
reported, not necessarily the dates on which infections occurred.

# Exercise 2: Analyzing Population Data (Project 9.10)

### Explore the dataset

I loaded population.json using Python's built-in json module. The top-level JSON structure is a list containing 152,361 individual
records. Each record is a dictionary with the following fields:

| Field | Description |
|---|---|
| id | Individual identifier |
| name | Individual's name |
| demographics.age | Age |
| demographics.household_income_band | Household income category |
| demographics.eyecolor | Eye color |
| last_measurements.weight | Weight |
| last_measurements.temperature | Temperature |


The demographics and last_measurements fields contain nested dictionaries. Therefore, age is accessed using record["demographics"]["age"], and weight is accessed using record["last_measurements"]["weight"].

Here is the example of the first record:
{
  "id": 1000000,
  "demographics": {
    "age": 88.89568973615539,
    "household_income_band": "< 35k",
    "eyecolor": "brown"
  },
  "name": "Edna Phelps",
  "last_measurements": {
    "weight": 67.12244970473641,
    "temperature": 37.45
  }
}
Top-level fields: ['demographics', 'id', 'last_measurements', 'name']
Demographic fields: ['age', 'eyecolor', 'household_income_band']
Measurement fields: ['temperature', 'weight']


### Analyze Age Distribution

I analyzed the age distribution and got the result in below: 
Mean age: 39.5105
Standard deviation: 24.1527
Minimum age: 0.0007
Maximum age: 99.9915

I explicitly compared histograms with 10, 30, and 100 bins. With 10 bins, the broad distribution was visible, but the wide intervals obscured the location of the drop in frequency around age 65. With 30 bins, this transition was clear and the histogram remained easy to interpret. With 100 bins, smaller fluctuations became visible, but they did not change the main interpretation.

<img width="1990" height="390" alt="image" src="https://github.com/user-attachments/assets/a22b736a-f33e-4772-9699-93545ba18336" />

I selected 30 bins because this setting clearly shows the major
change in frequency without adding unnecessary visual detail.

<img width="790" height="490" alt="image" src="https://github.com/user-attachments/assets/ab2db0ac-d690-4931-acda-22e5b091cd41" />

### Analyze Weight Distribution
Ages are approximately evenly distributed from 0 to around 65. The frequency drops sharply around age 65 and remains relatively flat at a lower level through approximately age 100.

The minimum age is close to zero, and the maximum is close to 100. Neither endpoint alone identifies an obvious anomalous record. The distribution is not a single bell-shaped distribution.

Below is the summary statistics:
| Statistic | Weight |
|---|---:|
| Mean | 60.8841 |
| Standard deviation | 18.4118 |
| Minimum | 3.3821 |
| Maximum | 100.4358 |

For making a bar graph to analyze weight distribution, I compared histograms with 10, 30, and 100 bins.

With 10 bins, the histogram showed a broad concentration around 60–80, but it concealed a narrow peak. With 30 bins, the peak became more apparent, although it was still combined with nearby values. With 100 bins, the narrow spike near 68 could be distinguished clearly from the surrounding broader distribution.

<img width="1489" height="390" alt="image" src="https://github.com/user-attachments/assets/ab0ffeaa-dc21-4f47-94d2-82eec1196271" />

I selected 100 bins because the finer intervals reveal an important feature that is obscured by wider bins. The large dataset provides enough observations to make this finer representation informative.

<img width="790" height="490" alt="image" src="https://github.com/user-attachments/assets/7fa86b0d-0190-4afa-8e69-04a7814d5387" />

Most weights are concentrated around 60–80, with a lower-frequency tail extending toward smaller values. The distribution contains a particularly sharp spike near 68.

A direct count showed that 22,613 records have a weight of exactly 68.0. This repeated value could reflect rounding, a default value, or the data-generation process. The file alone does not establish which explanation is correct.

The minimum and maximum weights are 3.3821 and 100.4358. These values should be interpreted in relation to age rather than automatically classified as anomalies.

## General Relationship

The scatterplot shows a strong positive relationship between age and weight from approximately age 0 to 20. After approximately
age 20, weights form a broad, mostly horizontal band. Thus, the relationship is not well described by one straight line across all ages. Weight increases strongly during younger ages and then approximately levels off in adulthood.

<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/e7ad7907-f844-4009-81d4-0bfa8cb0987b" />

A horizontal concentration at weight 68 is also visible, consistent with the repeated values identified in the weight analysis.

### Step-by-Step Anomaly Identification

1. I plotted all records with age on the horizontal axis and weight on the vertical axis.

2. I inspected the main pattern: younger individuals follow an increasing trend, while adults mostly occupy a higher weight band.

3. I noticed an isolated point near age 41 and weight 22, well below the main adult cluster.

4. Based on this visual observation, I filtered the original records for age greater than 20 and weight less than 30.

5. The filter returned exactly one record. I retrieved its name, identifier, age, and weight, then marked it on the scatterplot.

The thresholds were chosen to isolate the visually observed point. They are not a formal statistical or medical definition of an outlier.

Below is the identified individual:
Identified Individual

| Field | Value |
|---|---|
| Name | Anthony Freeman |
| ID | 1002902 |
| Age | 41.3 |
| Weight | 21.7 |

<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/4956c3d3-52b1-4029-b4a9-16098b10fa35" />

### Interpretation and Implications

Age 41.3 is ordinary when considered alone, and weight 21.7 occurs within the overall range of the dataset. However, their combination clearly deviates from the main adult age–weight pattern. This demonstrates why analyzing attributes jointly can reveal anomalies that separate summary statistics or histograms may miss.

The record should be flagged for verification. Possible explanations include a data-entry error, a unit mismatch, an incorrect linkage between records, or a genuine unusual case. The plot alone cannot determine the cause, so the record should not be automatically deleted or corrected without checking its source.


# Exercise 1: Efficiently Searching Patient Data (Project 9.17)

### patient ages distribution with different age bins

I tried different age bins and finally decide to use 17 bins to represent the whole histogram. Since the patient basically fall into 0-85 age range, if there is 17 bins, each bins will be around 5 years old. It is more reasonable. 

<img width="1790" height="490" alt="patient ages" src="https://github.com/user-attachments/assets/a097ac5d-3eed-4bfa-aceb-d3323bcbd382" />

### Duplicate Analysis
I used `collections.Counter` to count how many times each original age value occurred. I kept the ages as floating-point values because the XML file stores ages with decimal precision. Converting them to integers would remove information and could incorrectly classify different ages as duplicates. Below is the analysis output:

Total number of patients: 324357
Number of unique age values: 324357
Number of duplicated age values: 0

No patients share the exact same age.

### Reflection
A standard binary search can determine whether a target age exists in a sorted list in O(log n) time. If duplicate values are present, however, the algorithm may return any one of the matching positions. It does not necessarily identify the first or last occurrence.

This difference matters when the goal is to find all patients with a particular age. Finding any match only confirms that the age exists, whereas finding the left and right boundaries identifies the complete range of matching records.

I will address this by sorting the ages and using left and right boundary searches. The left-boundary search identifies the first position at which the target could occur, while the right-boundary search identifies the position immediately after the final occurrence. The number of matches can therefore be calculated as `right - left`.

The provided dataset contained no exact duplicate ages, so the first and last matching positions are identical for every age in the dataset. Nevertheless, the boundary-based logic would continue to work correctly if duplicate values were present.

### Gender Distribution
Gender is encoded as an attribute of each `<patient>` element in the XML file. For example:

```xml
<patient age="19.529988374393394"
         gender="female"
         name="Tammy Martin">
</patient>
```

First 10 gender values: ['female', 'female', 'male', 'male', 'male', 'female', 'female', 'female', 'female', 'male']
Distinct gender categories: ['female', 'male', 'unknown']
Gender counts:
female: 165293
male: 158992
unknown: 72

<img width="790" height="490" alt="patient gender distribution" src="https://github.com/user-attachments/assets/7b69c7ca-d519-4b97-af9c-2611f9554ed1" />

### Age Sorting & Top K Retreival 
Sort patients based on their age. And here's the result: Oldest patient:
Name: Monica Caponera
Age: 84.99855742449432
Gender: female

Top 10 oldest patients:
1. Monica Caponera, age=84.9986, gender=female
2. Raymond Leigh, age=84.9983, gender=male
3. Tracy Walker, age=84.9982, gender=female
4. Michael Ali, age=84.9979, gender=male
5. Stephan Yeargin, age=84.9972, gender=male
6. Caprice Medina, age=84.9965, gender=female
7. Michael Bowden, age=84.9964, gender=male
8. Helen Guest, age=84.9962, gender=female
9. Elizabeth Jackson, age=84.9962, gender=female
10. Agnes Weaver, age=84.9958, gender=female


### Algorithmic Trade-offs: Finding the Second-Oldest Patient
To find the second-oldest patient without sorting the entire dataset, I used a single-pass algorithm. The algorithm maintains two variables: `oldest` and `second_oldest`.

For each patient, the algorithm first compares the patient's age with the current oldest age. If the new patient is older, the previous oldest patient becomes the second-oldest, and the new patient becomes the oldest. Otherwise, the patient's age is compared with the current second-oldest age and replaces it when appropriate.

The resulting patients were:

- Oldest: Monica Caponera, age 84.99855742449432
- Second-oldest: Raymond Leigh, age 84.9982928781625

The single-pass approach examines each of the n patient records once, so its time complexity is O(n). It stores only two additional patient references, giving it O(1) auxiliary space complexity.

In comparison, sorting all patient records by age requires O(n log n) time and generally O(n) space for a newly sorted list. After sorting, however, retrieving a patient at a known rank, such as the oldest or second-oldest patient, takes O(1) time through list indexing.

The single-pass method is preferable when only the oldest, second-oldest, or a small number of extreme values are needed once. It avoids the additional time and memory required to create a complete ordering.

Upfront sorting is more useful when the ordered dataset will be reused for many ranking queries, such as retrieving several different ranks,
displaying all patients in age order, or repeatedly obtaining different top-k groups. In that case, the initial O(n log n) sorting cost may be worthwhile because subsequent index-based lookups take O(1), while retrieving the first k records takes O(k).

### Binary Search for Specific Target Age
At each iteration, the algorithm compared 41.5 with the age of the patient at the middle index of the current search range. If the middle
age was equal to 41.5, the algorithm returned that patient's index. If the middle age was greater than 41.5, the algorithm continued searching
to the right because younger patients appear later in the descending list. If the middle age was less than 41.5, it searched to the left because older patients appear earlier in the list.

The algorithm found one exact match:

- Name: John Braswell
- Age: 41.5
- Gender: male

If the target age falls between two existing age values, the search range eventually becomes empty, meaning that `left` becomes greater than `right`. The function then returns `-1` to show that no exact match was found. The algorithm does not return the nearest age because the task requires a patient whose age is exactly equal to the target.

If multiple patients have the same target age, the current implementation returns whichever matching record it encounters first during the binary search. This record is not guaranteed to be the first or last matching patient in the sorted list. To retrieve every matching patient, the algorithm could be extended to find the boundaries of the duplicate values: one search would continue to the left after finding a match, and another would continue to the right. All records between those two boundary indices would have the target age.

In this dataset, only one patient has an age of exactly 41.5, so the current implementation returns the only matching patient. Binary search takes O(log n) time because each comparison eliminates approximately half of the remaining search range. This is more efficient than a linear search taking O(n) time, provided that the patient records have already been sorted.

### Counting Patients Above an Age Threshold

The patient records were previously sorted by age in descending order, from oldest to youngest. Binary search located the patient whose age was
exactly 41.5 at index 150,470.

Because the records are stored in descending order, every patient before this index is older than 41.5, and the patient at the returned index is exactly 41.5. Therefore, all patients from index 0 through index 150,470 satisfy the condition `age >= 41.5`.

Since Python uses zero-based indexing, the number of patients in this
range is:

`index + 1 = 150,470 + 1 = 150,471`

Therefore, 150,471 patients are at least 41.5 years old.

The binary search requires O(log n) time because it eliminates approximately half of the remaining search range at each iteration.
After the matching index has been found, calculating `index + 1` requires only one arithmetic operation and therefore takes O(1) time.
This avoids performing an additional O(n) traversal to count all qualifying patients.

This direct calculation works because the records are sorted and 41.5 occurs only once in this dataset. If multiple patients had the same target age, a standard binary search could return any matching index. In that situation, the search would need to find the last occurrence of 41.5 in the descending list. The count of patients with ages greater than or equal to 41.5 would then be `last_matching_index + 1`.

### Age Range Queries

I implemented a range-query function that counts patients satisfying:

`low_age <= age < high_age`

The function reuses the patient records previously sorted by age in descending order. It performs two boundary binary searches rather than iterating through every patient.

The first search finds the index of the first patient whose age is strictly less than `high_age`. This is the beginning of the requested
range because patients whose ages equal the upper bound must be excluded.

The second search finds the index of the first patient whose age is strictly less than `low_age`. This is the position immediately after the requested range because patients whose ages equal the lower bound must be included.

All qualifying patients appear consecutively between these two
indices. Therefore, the result can be calculated as:

`end_index - start_index`

### Test Cases

| Range | Count | Purpose |
|---|---:|---|
| `40 <= age < 50` | 42,525 | Tests a normal range within the dataset |
| `41.5 <= age < 42` | 2,232 | Tests a lower bound equal to an existing age |
| `41.5 <= age < 41.5001` | 2 | Tests a very narrow range |
| `85 <= age < 100` | 0 | Tests a range above all existing ages |
| `-10 <= age < 0` | 0 | Tests a range below all existing ages |
| `-1 <= age < 100` | 324,357 | Tests a range containing the full dataset |
| `50 <= age < 50` | 0 | Tests an empty range |
| `60 <= age < 50` | 0 | Tests invalid reversed boundaries |
| `84.99855742449432 <= age < 100` | 1 | Tests a lower bound equal to the maximum age |

The function returns zero when `low_age >= high_age` because no value can satisfy an empty or reversed interval.

Each boundary search takes O(log n) time because it eliminates approximately half of the remaining search interval during each
iteration. The algorithm performs two binary searches, so its total search time is:

`O(log n) + O(log n) = O(log n)`

After the two boundary indices have been identified, subtracting them takes O(1) time. The algorithm does not iterate through the patients inside the requested range, so its execution time does not depend on the number of matching patients.

The boundary-search approach also handles duplicate ages correctly. All patients equal to `low_age` are included, while all patients equal
to `high_age` are excluded, matching the required half-open interval `[low_age, high_age)`.

### Multi-Criteria Age and Gender Range Queries

To support age-range queries that also count male patients, I combined the age-sorted patient records with a prefix-sum index for gender.

The records were already sorted by age in descending order. I created a list called `male_prefix`, where `male_prefix[i]` stores the number of male patients among the first `i` records. The prefix list begins with zero and contains one additional element beyond the number of patient records.

During the one-time setup, each patient was examined once. If the patient's gender was encoded as `male`, the next prefix value was increased by one. For `female` and `unknown` patients, the previous prefix value was repeated.

For each query, I first used two boundary binary searches to locate the records satisfying:

`low_age <= age < high_age`

The total number of patients was calculated as:

`end_index - start_index`

The number of male patients was calculated as:

`male_prefix[end_index] - male_prefix[start_index]`

The prefix subtraction works because the first prefix value counts all male patients before the end of the range, while the second counts all male patients before the beginning of the range. Subtracting them leaves only the male patients within the requested interval.

### Test Cases

| Age range | Total patients | Male patients |
|---|---:|---:|
| `40 <= age < 50` | 42,525 | 20,873 |
| `41.5 <= age < 42` | 2,232 | 1,135 |
| `85 <= age < 100` | 0 | 0 |
| `-1 <= age < 100` | 324,357 | 158,992 |
| `50 <= age < 50` | 0 | 0 |
| `84.99855742449432 <= age < 100` | 1 | 0 |

Constructing the prefix-sum list requires O(n) time and O(n) additional space, but this initialization is performed only once.

Each query then performs two binary searches, requiring O(log n) time. The total count and male count are both calculated with constant-time indexing and subtraction, requiring O(1) additional time. Therefore, the overall time complexity of each query after setup is O(log n).

Without the prefix-sum index, the algorithm would need to inspect every patient within the selected age range to count male patients. In the worst case, this could require O(n) time. The prefix-sum structure avoids that scan and preserves sub-linear query performance.

# Exercise 2: Algorithm Analysis & Performance Measurement (Project 9.10)

### 2a. Functionality Identification & Conceptual Mechanics
Both alg1 and alg2 sort the input in ascending order. The examples show that both functions handle unsorted, already sorted, and reverse-sorted lists. They also preserve duplicate values, correctly order negative numbers, and return empty or single-element lists unchanged. Below is the example output: 

| Input | `alg1` Output | `alg2` Output |
|---|---|---|
| `[6, 8, 3, 1]` | `[1, 3, 6, 8]` | `[1, 3, 6, 8]` |
| `[1, 2, 3, 4]` | `[1, 2, 3, 4]` | `[1, 2, 3, 4]` |
| `[5, 4, 3, 2]` | `[2, 3, 4, 5]` | `[2, 3, 4, 5]` |
| `[1, 1, 4, 2]` | `[1, 1, 2, 4]` | `[1, 1, 2, 4]` |
| `[-7, 7, -2, 0]` | `[-7, -2, 0, 7]` | `[-7, -2, 0, 7]` |
| `[]` | `[]` | `[]` |
| `[9]` | `[9]` | `[9]` |

#### How alg1 Works: Bubble Sort
alg1 repeatedly scans the list from left to right. It compares adjacent elements and swaps them if the left element is larger than the right element. It stops when a complete scan produces no swaps.
For the input [6, 8, 3, 1], the scans proceed as follows:
| Scan | List After the Scan | Explanation |
|---|---|---|
| 1 | `[6, 3, 1, 8]` | `8` swaps with `3`, then with `1`, moving to the end. |
| 2 | `[3, 1, 6, 8]` | `6` swaps with `3`, then with `1`. |
| 3 | `[1, 3, 6, 8]` | `3` swaps with `1`, completing the sorting. |
| 4 | `[1, 3, 6, 8]` | No swaps occur, so the function stops. |

Larger elements can move several positions to the right during one scan, while a smaller element can move only one position to the left per scan. This explains why reverse-sorted input such as [5, 4, 3, 2] requires multiple scans. For the already sorted input [1, 2, 3, 4], the first scan produces no swaps, so the function stops immediately. Equal adjacent elements, such as the two 1s in [1, 1, 4, 2], are not swapped because the comparison uses a strict inequality.

#### How alg2 Works: Merge Sort
alg2 recursively divides the list into two halves until each sublist contains at most one element. It then merges the sorted sublists by repeatedly selecting the smaller of their leading elements.
For the input [6, 8, 3, 1], the process is:
| Stage | Operation | Result |
|---|---|---|
| Divide | Split the original list into two halves. | `[6, 8]` and `[3, 1]` |
| Divide again | Split each half into single-element lists. | `[6]`, `[8]`, `[3]`, `[1]` |
| Merge each half | Combine the single-element lists in ascending order. | `[6, 8]` and `[1, 3]` |
| Final merge | Compare the leading elements of both sorted halves. | `[1, 3, 6, 8]` |

During the final merge, 1 is selected before 6, followed by 3. The right half is then exhausted, so the remaining left-half elements, [6, 8], are appended. Unlike alg1, alg2 still divides and merges an already sorted list such as [1, 2, 3, 4]. Duplicate values are preserved: when two leading values are equal, this implementation selects the right-hand value first, but both values remain in the output. Empty and single-element lists are returned immediately because they already satisfy the base case.

### 2b. Benchmarking Strategy and Log-Log Scaling Analysis
Benchmarking Methodology
Nine approximately logarithmically spaced input sizes between 100 and 4,000 were generated using np.logspace().

For each input size and data generator, the input sequence was generated before timing began. Both algorithms processed the same sequence. Execution time was measured using time.perf_counter(), excluding data generation and correctness checks.

Each function was run once before measurement, then timed five times. The median of those five measurements was used to reduce the influence of occasional timing fluctuations.

Separate log-log plots were generated for data1, data2, and data3. Each plot compares the execution times of both algorithms. Slopes were estimated by fitting a straight line to the logarithms of the largest five input sizes and their corresponding execution times.

Empirical Results
At n = 4000, the measured median execution times were:
| Dataset | `alg1` Time (seconds) | `alg2` Time (seconds) | Faster Function |
|---|---:|---:|---|
| `data1` | 0.900950 | 0.005193 | `alg2` |
| `data2` | 0.000160 | 0.005185 | `alg1` |
| `data3` | 1.216347 | 0.005291 | `alg2` |

The fitted log to loge slope were:
| Dataset | `alg1` Slope | `alg2` Slope |
|---|---:|---:|
| `data1` | 2.281 | 1.102 |
| `data2` | 1.045 | 1.147 |
| `data3` | 2.126 | 1.149 |

Interpreting Log-Log Slopes
If execution time approximately follows a power law,\[T(n) = Cn^p,\]taking logarithms gives:\[\log T(n) = \log C + p\log n.\]
Therefore, the slope on a log-log plot estimates the growth exponent \(p\). A slope near 1 suggests approximately linear growth, while a slope near 2 suggests approximately quadratic growth.

For alg1, the slope on sorted input was close to 1, consistent with its best-case complexity of \(O(n)\). Its slopes on data1 and reverse-sorted input were close to 2, consistent with approximately quadratic behavior over the tested range.

For alg2, the theoretical complexity is \(O(n\log n)\), which is not a pure power law. Over a finite range, this commonly produces a log-log slope slightly greater than 1. The measured slopes of approximately 1.10–1.15 are consistent with this behavior.
Empirical slopes do not prove asymptotic complexity. For example, the slope of 2.281 for alg1 on data1 does not imply a theoretical complexity of \(O(n^{2.281})\). Changes in input structure, finite input sizes, and timing variability can cause fitted slopes to differ from theoretical expectations.

### 2c. Evaluation and Data-Dependent Behavior

data1: A Sequence with Chaotic Dynamics
data1 generates values through numerical updates of the Lorenz system. With fixed parameters and initial conditions, the sequence is deterministic rather than independently random. Neighboring values are related, and some local sections may be increasing or decreasing, but the longer sequence is not globally sorted.

For alg1, this disorder generally requires many scans and swaps. Small elements that belong far to the left are particularly expensive because they can move left by only one position per scan. The measured runtime showed approximately quadratic growth over the tested range.

For alg2, input order does not change the basic recursive division structure. The function continues to divide the sequence into halves and merge sorted sublists, maintaining \(O(n\log n)\) complexity. At n = 4000, alg2 was approximately 173 times faster than alg1.

data2: Already Sorted Input
data2(n) generates [0, 1, ..., n - 1], which is already sorted in ascending order.

For alg1, the first scan produces no swaps, so the function immediately exits. Including the initial list copy, its execution time is \(O(n)\).

alg2 does not check whether the complete input is already sorted. It still performs recursive splitting and merging. Although sorted input can reduce some merge comparisons, slicing and copying elements still require work, so the overall complexity remains \(O(n\log n)\). At n = 4000, alg1 was approximately 32 times faster than alg2.

data3: Reverse-Sorted Input
data3(n) generates [n, n - 1, ..., 1], which is ordered in the opposite direction from the required output.

For alg1, the smallest element must move from the last position to the first. Because it moves left by only one position per scan, this requires n - 1 scans. One additional scan with no swaps confirms completion.For n ≥ 2, this implementation therefore performs n scans, each containing n - 1 comparisons. The total number of comparisons is:\[n(n - 1).\]This gives \(O(n^2)\) time complexity. Reverse-sorted input also requires many swaps, making it a worst-case input for alg1.

For alg2, reverse ordering changes which sublist supplies elements during merging, but it does not change the number of recursive levels or the overall amount of work per level. Its complexity remains \(O(n\log n)\). At n = 4000, alg2 was approximately 230 times faster than alg1.

Here is the generated plot for the final result:
<img width="1590" height="494" alt="image" src="https://github.com/user-attachments/assets/f5d21648-fe57-414d-8a83-79492ca27e92" />

Algorithm Selection
| Input Condition | Preferred Function | Reason |
|---|---|---|
| Known to be sorted in ascending order | `alg1` | Completes after one scan with no swaps, taking \(O(n)\) time. |
| Unordered or unknown input order | `alg2` | Provides \(O(n\log n)\) scaling and handles large inputs more efficiently. |
| Reverse-sorted input | `alg2` | Avoids the quadratic scans and numerous swaps required by `alg1`. |
| Nearly sorted input | Depends on element displacement and input size | `alg1` can finish quickly if few scans are needed, but severely misplaced elements can still cause quadratic behavior. |

Nearly sorted input does not always guarantee good performance for alg1. For example, [2, 3, ..., n, 1] has only one obviously misplaced element, but moving 1 to the beginning requires approximately n scans. This still results in \(O(n^2)\) execution time.

The empirical plots and algorithm mechanics support choosing alg2 for large inputs with unknown or disordered structure. For input known to be already sorted, the early stopping behavior of alg1 makes it faster in this experiment.

### A3: Approximate Membership, Reconstruction Risks, and Trade-offs in Health Data Structures 

## Reconstruction Analysis

Although the Bloom filter’s bit vector does not store raw words, ASCII values, or string pointers, it still reveals information about the vocabulary through membership queries. An adversary can systematically generate plausible candidate words—for example, all single-character substitutions of a target such as `floeer`—and submit each candidate to the Bloom filter. If any required bit is zero, the candidate is definitely not in the protected vocabulary. If every required bit is one, the candidate may be present. Repeating this procedure over many carefully constructed mutations allows the adversary to identify likely vocabulary entries without directly accessing the original word database.

This process constitutes data leakage because the Bloom filter acts as a probabilistic membership oracle. Its responses expose information derived from the protected vocabulary, even though the vocabulary itself is not stored in readable form. Some accepted candidates may be false positives caused by hash collisions, so the reconstruction is not always exact. However, an adversary can combine the Bloom filter’s responses with linguistic knowledge, dictionaries, or repeated mutation queries to distinguish plausible words from random strings. Therefore, a Bloom filter should be treated as a sensitive representation of its input data rather than as an anonymized or cryptographically protected version of the database.

## Execution Test and False-Positive Analysis

I loaded the 466,550 entries from `words.txt` into a Bloom filter containing `10**7` bits. Since eight bits are stored in one byte, the bit vector requires approximately:

```text
10,000,000 / 8 = 1,250,000 bytes ≈ 1.25 MB
```

I then called `suggest_corrections()` with the misspelled word `floeer`. The function generated every possible single-character substitution and retained the candidates accepted by the Bloom filter.

Using only the first hash function produced:

```text
['bloeer', 'qloeer', 'fyoeer', 'flofer',
 'floter', 'flower', 'floeqr', 'floees']
```

Using the first and second hash functions produced:

```text
['fyoeer', 'floter', 'flower']
```

Using all three hash functions produced:

```text
['floter', 'flower']
```

These results demonstrate how increasing the number of hash functions reduces false positives. With one hash function, a candidate is accepted whenever its single corresponding bit is already set. Because many words can map to the same bit, pseudo-matches such as `bloeer`, `qloeer`, `flofer`, `floeqr`, and `floees` are incorrectly accepted. These candidates do not necessarily appear in the original vocabulary; their bit positions may have been set by other words.

With two hash functions, a candidate must map to two occupied bit positions. It is less likely for a nonmember to satisfy both conditions accidentally, so most of the pseudo-matches disappear. However, `fyoeer` remains because both of its required bits happen to be set. With all three hash functions, the candidate must pass three separate bit checks. `fyoeer` fails the third check and is removed, leaving only `floter` and `flower`. In this case, `floter` is a valid Middle English word, while `flower` is the intended correction.

Therefore, the additional hash functions make a positive result more selective: a false candidate must collide at multiple bit positions instead of only one. This lowers the false-positive rate in this experiment while preserving the genuine suggestions. However, more hash functions do not always improve a Bloom filter indefinitely. In a fixed-size filter, each additional hash function sets more bits during insertion, causing the bit vector to become saturated more quickly. The number of hash functions must therefore be balanced against the filter size and the number of stored elements.


###  Part 2 Analysis： Filter Capacity Threshold

Increasing the Bloom filter size generally reduced the Misidentified Percent and increased the Good Suggestion Percent. With three hash functions, the Good Suggestion Rate reached 84.22% at 10,000,000 bits, so approximately 10–11 million bits would be needed to reach 85%. The one- and two-hash configurations did not reach 85% within the tested range. At 10,000,000 bits, they reached only 0.33% and 54.45%, respectively, so both require more than 10 million bits. Based on the observed trends, two hashes would likely require moderately more than 10 million bits, whereas one hash would require a substantially larger filter. Exact thresholds for these configurations cannot be determined without testing larger filter sizes.

| Hash functions | Good Suggestion Rate at 10M bits | Approximate size for 85% |
|---:|---:|---:|
| 1 | 0.33% | More than 10M bits; far beyond the tested range |
| 2 | 54.45% | More than 10M bits |
| 3 | 84.22% | Approximately 10–11M bits |

The results also show that more hash functions do not always improve performance when the filter is very small. At 1,000,000 bits, the three-hash filter had a 42.66% misidentification rate, which was higher than the one- and two-hash results. This occurs because setting three bits per word saturates a small filter more quickly.

### 2. Combinatorial Explosion and Mutation Spaces

For a word of length \(L\) over the 26-letter English alphabet, single-character substitution generates at most \(25L\) changed candidates because each position can be replaced by 25 other letters. This is a relatively small search space. If the lookup is expanded to edit distance 2 and includes substitutions, insertions, deletions, and transpositions, the number of candidates grows approximately quadratically with word length. Every candidate produced by the first edit can itself receive another edit, creating on the order of \(O(L^2A^2)\) possibilities, where \(A\) is the alphabet size.

This combinatorial growth would increase the number of Bloom-filter queries and the number of false-positive suggestions. Even if the false-positive probability for one query remains unchanged, performing many more queries increases the expected total number of false positives. The suggestion list would therefore be more likely to exceed the three-suggestion limit, reducing the Good Suggestion Rate. A similar problem occurs in bioinformatics when matching sequence reads or \(k\)-mers containing sequencing errors. Allowing multiple substitutions, insertions, or deletions creates a rapidly expanding neighborhood of possible sequences. Practical sequence-matching systems therefore often use strategies such as seed-and-extend, minimizers, or specialized approximate-matching algorithms instead of enumerating every possible mutation.

### 3. Cryptographic vs. Non-Cryptographic Hash Efficiency

The starter code uses cryptographic hash functions such as SHA-256, BLAKE2b, and SHA3-256. These functions perform many rounds of mixing and operate on relatively large internal states because they are designed to provide security properties such as pre-image resistance and strong collision resistance. These operations make them computationally more expensive than necessary for repeatedly inserting and querying millions of values in a Bloom filter.

Non-cryptographic hash functions such as MurmurHash3, xxHash, and FNV-1a are designed for speed and good statistical distribution rather than cryptographic security. A Bloom filter mainly requires hash outputs that are fast, approximately uniform, and sufficiently independent. It does not normally need pre-image resistance because the Bloom filter is not intended to encrypt or hide its inputs. It also does not need cryptographic collision resistance because Bloom filters already permit collisions and false positives by design.

Therefore, a standard Bloom filter gains little structural or security benefit from cryptographic hashes. Cryptographic hashing does not prevent vocabulary reconstruction through repeated membership queries, because an adversary who knows the hash functions can hash the same candidates. Non-cryptographic hashes are generally more appropriate for large-scale Bloom-filter operations because they provide the required distribution at much lower computational cost.

<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/507a4dfc-edc7-49e8-9884-bacdae8fa25c" />

