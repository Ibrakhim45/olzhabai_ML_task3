1.The goal. Why do we need a model? 
The bank is conducting an advertising campaign and wants to sell term deposits through a phone call. 
Our goal is not to guess at random, but to sort it so that we can call 10% of our customers and get maximum sales at minimum cost. 
Only 11.3% of people agree with us. If the model always answers "no", the accuracy will be 88.7%, because of this, the usual accuracy does not work and the bank will not sell anything at all. Therefore, we are looking at the ranking metrics - ROC-AUC and Average Precision.

2.What's with the data.
train.csv - 32,940 customers , test.csv - 8,236

The duration column was removed. Because we will only know it after the operator completes the call. If you leave her in training, the model will look into the future.

Omissions and stubs. The values of "unknown" in several columns job, education, default and others were decided to remain a separate category.

In "pdays", the value 999 means "the customer has not been called before" (96.2% clients). We replaced 999 with the median and added the a 0/1 flag “contacted before”.

3.How the quality was checked (Validation and Pipeline)
, 20% of the data of 6,588 people were set aside. The model didn't see them until the finale.

The remaining 80% raced on 5-fold Stratified Cross-Validation

The standard deviation of the folds was about 0.007–0.010. This means that if one model is ahead of the other by less than 0.010, we consider them to be the same, and the difference is random noise.
All the preprocessing was strictly inside the Pipeline. No data from validation was included in the training.

4.Results and model selection
We compared three algorithms on the same 5 folds:

Baseline (Dummy): ROC-AUC = 0.5000
Decision tree (max_depth=8): ROC-AUC = 0.7754
Random forest (max_depth=10): ROC-AUC = 0.7928
Logistic regression (C=0.1): ROC-AUC = 0.7854

Why choose Logistic Regression:
Although the Random Forest showed 0.7928, the difference with the 0.7854 regression was only 0.0074. This is less than the noise between folds ± 0.010. There is no point in complicating the project for the sake of a phantom advantage, so we took a simple, fast and understandable logistic regression. 
The class_weight='balanced' weighting added only +0.0006 to the ROC-AUC, so we abandoned it.

5.Totals based on deferred data 
We ran a regression on those 20% of customers who were postponed at the very beginning:
ROC-AUC: 0.8167
Average Precision: 0.4927
The cut-off threshold of the top 10% is 0.2878 (we call 659 clients out of 6 588).

What does a call center manager get?:
Precision = 51.1%: Every second call from the selected 10% ends with the purchase of a deposit of 337 sales for 659 calls.
Recall = 45.4%: After calling only a tenth of the database, we collect almost half of all possible sales.
Lift = 4.54: The model works 4.54 times more efficiently than if the operators called people at random.

6.Where the model can "stumble"
A shift in the economy: The data was shared randomly, so the training and the test have the same macro indicators of the euribor3m rate, etc. In real life, the economy is changing, and the quality of future data will drop.
Types of communication: Cellular contacts convert at 14.7 % versus 5.3 % for telephone. If the structure of the base changes, the scores will move out.

7.What to monitor after launch.
Check the conversion rate of calls every week: Lift should stay well above 1. On my test it was about 4.5, so a lower value in real use is normal.
Follow the drift of economic signs euribor3m, nr.employed.
Retrain the model on fresh calls every 3-6 months.