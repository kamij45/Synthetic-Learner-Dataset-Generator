# Synthetic-Learner-Dataset-Generator
A tool that generates realistic but entirely fake student exam data (online or physical exams, Pass/Fail outcome), then measures how realistic and how private that data is, without ever using real student records.

# Table Of Contents
1. Overview
2. Challenge
3. Features
4. Project Structure
5. Dataset Schema
6. Quality Metrics and Results
7. Privacy Checks
8. Testing
9. Limitations
10. Responsible Use
11. Author

# Overview
Problem: Research teams need safe, realistic test data. Real student records are private and cannot be shared freely, but pipelines, dashboards and prediction models still need data to be built and tested.

Solution: This project generates synthetic learner records for an exam setting. Each record describes a student's study habits, attendance, screen time, sleep and internet quality, plus an exam score. The exam score is the final screening: a student passes if the 
score reaches the pass mark (50 by default), otherwise they fail.

The project has four parts:
# Parts           |     What it Delivers
------------------|-----------------------------------------------------------------------------------------------
1.	Schema design | 	A written blueprint of every column plus a validator that lists problems and suggested fixes
2. Data generator	|   Two independent generators: rule-based and seed-based
3. Quality metrics|   Statistics, distribution charts, a model-based usefulness test and a cross-test
4	Privacy checks	|   Duplicate, copy, near-copy, uniqueness and identifier checks with an issue list

# Challenge
Evaluating realism without copying real student records.
Normally you would prove fake data is realistic by comparing it with real data. That is not allowed here. Instead, realism is judged indirectly:
1. Designed patterns appear. If study hours are meant to raise scores, they should.
2. Values are believable. Everything stays inside valid ranges and the pass rate is plausible.
3. A model can learn from it. A classifier trained on the data should predict Pass/Fail well (usefulness).
4. Independent generators agree. Two generators built in different ways should identify the same main drivers.
5. Nothing is copied. Privacy checks confirm no generated row duplicates or sits suspiciously close to a seed row.

# Features
1. Two generators built on different ideas, so they can be compared
2. Fully reproducible: every generator takes a random seed
3. Explicit schema with a validator that returns an issue list with suggested corrections
4. Quality evaluation with uncertainty (5-fold cross-validation, reported as mean ± spread)
5. Cross-test between datasets to check that they agree on the patterns
6. Protection against data leakage (the exam score is never a model input)
7. Privacy checks: exact copies, near-copies, uniqueness (re-identification risk) and identifier scan
8. Automated tests, including privacy checks that must catch planted problems

# Project structure
synthetic-learner-dataset-generator/
├── Synthetic_Learner_Dataset_Generator.ipynb   # the full workflow, run top to bottom
├── my_seed.csv                                 # hand-made seed rows (invented, not real)
├── synthetic_students.csv                      # output of the rule-based generator
├── my_synthetic_students.csv                   # output of the seed-based generator
├── README.md                                   # this file
└── requirements.txt                            # numpy, pandas, scikit-learn, matplotlib


# Dataset Schema
Nine columns per student:
# Column       	        Type      	        Range        	           Meaning
1.ID	                  Text	           S0001, S0002,  	      Made-up identifier, never a real student number
2.Age	                  Integer	         18 to 30	              Student age
3.Study_hrs	            Float	           0 to 12	              Study hours per Day
4.Attendance%	          Float	           0 to 100	              Percentage of classes attended
5.Total_screen_time   	Float	           0 to 24	              Screen hours per day
6.Sleep_hrs	            Float	           0 to 24	              Sleep hours per night
7.Internet_quality	    category	    Poor, Average, Good	      Quality of the student's internet connection
8.exam_score	          Float            0 to 100	              Score in the exam
9.Result	              category	     Pass, Fail	              Pass if exam_score ≥ 50, otherwise Fail


# Generator A: rule-based
Patterns are rules chosen by hand, each controlled by a number in CONFIG, so every assumption can be explained and changed.

1.Factor	Effect on exam score
2.Study hours	+1.3 marks per weekly hour
3.Attendance	+0.30 marks per percentage point
4.Sleep	Loses 1.5 marks per hour away from 7.5 hours
5.Screen time	Loses 0.9 marks per daily hour above 6
6.Internet quality	+1.5 marks per step on a 1 to 5 scale
7.Random noise	Normal, standard deviation 8, so real-life unpredictability is kept

Attendance is also made to rise slightly with study hours, and the final score is limited to 0 to 100. 

# Generator B: seed-based
1.You write about 25 invented rows by hand in Excel (my_seed.csv).
2.The code measures the average and relationships (covariance) between the numeric columns.
3.It draws new rows from a multivariate normal distribution with those properties, then cleans values to valid ranges.
4.Internet_quality is a category, so it is sampled separately using the label proportions found in the seed rows.
5.Result is calculated from exam_score with the pass mark.

# Quality metrics and results
Side-by-side averages
Column	          Rule-based	       Seed-based
Age	                21.9	                22.7
Study_hrs	          13.9          	      6.8
Attendance%	        69.3	                51.0
Total_screen_time	  7.3	                  9.2
Sleep_hrs        	  6.7	                  5.7
exam_score	        56.8	                56.6


# Usefulness: can the model predict Pass/Fail? (5-fold cross-validation)
The exam score is not used as an input, because Result is calculated from it and the model would simply learn the threshold (data leakage).
 Dataset	     Model	               Accuracy	   F1     ROC-AUC
1.Rule-based	Logistic regression    	0.73	     0.81  	0.77 
2.Rule-based	Random forest	          0.69       0.78  	0.76 
3.Seed-based	Logistic regression	    0.85       0.89  	0.92 
4.Seed-based	Random forest	          0.84       0.88  	0.89 

# Privacy checks
Each dataset is checked for:
 Check	             What it looks for	            
1.Duplicates          Repeated IDs or repeated rows (ignoring ID)	
2.Exact seed copies	  Generated rows identical to a seed row	
3.Near-copies	        Nearest-neighbour distance to a seed row in standardised units	
4.Uniqueness	        Rows in groups smaller than k = 3 when described by age band, study-hours band and internet quality
5.Direct identifiers	Emails, phone numbers, and IDs that do not look like S0001\

# Limitations
1. A small seed file (around 25 rows) gives noisy patterns in the seed-based generator.
2. The seed-based generator assumes numeric columns are roughly bell-shaped; real data is rarely that tidy.
3. Realism is judged indirectly; there is no real-data benchmark to compare against.
4. The data is simplified: it has no teachers, courses, time dimension or free-text feedback.

# Testing
The notebook ends with a test cell covering the core flows:
1.Schema: nine columns in the right order, valid ranges, valid labels, Result consistent with the pass mark
2.Reproducibility: the same seed gives identical data and a different seed gives different data
3.Leakage: exam_score is not a model input
4.Usefulness: the model beats a "predict the majority" baseline
5.Privacy: clean data raises no issues, and planted problems are caught (duplicate ID, exact seed copy, near-copy, email)

# Responsible use
All data in this project is synthetic. It is meant for testing pipelines and tools, not for making decisions about real students. Do not treat patterns in it as evidence about real learners.

# Author
[Your name] [Your GitHub link] · [Your LinkedIn link]
