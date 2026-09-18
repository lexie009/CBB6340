## CBB6340

# Daily New COVID-19 Cases (Project 9.10)

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


# Efficiently Searching Patient Data (Project 9.17)

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

