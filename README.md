# 🎓 Springboard Data Science Bootcamp

Projects and case studies from my Springboard Data Science Career Track.

## Capstone Two: Predicting Wafer Failures from Process Sensors

Can in-line sensor readings flag a wafer that will fail final test? Using the [UCI SECOM](https://archive.ics.uci.edu/dataset/179/secom) dataset (1,567 units, 590 sensors, 6.6% failures), I compared Logistic Regression, Random Forest and LightGBM with time-ordered validation. Logistic Regression was the only model that held up on later wafers (test PR AUC 0.166 against a 0.054 random baseline, 5 of 17 failures caught).

- [Project README](capstone_two/README.md)
- [Final report (PDF)](capstone_two/Capstone_Two_Final_Project_Report.pdf)
- [Final slides (PDF)](capstone_two/Capstone_Two_Final_Slides.pdf)

## Repository structure

```
capstone_two/    Capstone Two project: notebooks, data, outputs, report and slides
notebooks/       Guided case studies (regression, tree models, boosting, clustering, time series and more)
SQLFilesTier2/   SQL case study (SQLite database and tasks)
story_telling/   Data storytelling presentation
```

## Tools

Python, pandas, scikit-learn, LightGBM, XGBoost, matplotlib, SQL (SQLite)

## Author

Thomas Truong
