# timechop
Generate temporal validation time windows for matrix creation


[![Build Status](https://travis-ci.org/dssg/timechop.svg?branch=master)](https://travis-ci.org/dssg/timechop)

[![codecov](https://codecov.io/gh/dssg/timechop/branch/master/graph/badge.svg)](https://codecov.io/gh/dssg/timechop)

[![codeclimate](https://codeclimate.com/github/dssg/timechop.png)](https://codeclimate.com/github/dssg/timechop)

In predictive analytics, temporal validation can be complicated. There are a variety of questions to balance: How frequently to retrain models? Should the time between rows for the same entity in the train and test matrices be different? Keeping track of how to create matrix time windows that successfully answer all of these questions is difficult. 

That's why we created timechop. Timechop takes in high-level time configuration (e.g. lists of train label spans, test data frequencies) and returns all matrix time definitions. 


Timechop currently works with the following:

- feature_start_time - data aggregated into features begins at this point
- feature_end_time - data aggregated into features is from *before* this point
- label_start_time - data aggregated into labels begins at this point
- label_end_time - data aggregated is from *before* this point
- model_update_frequency - amount of time between train/test splits
- training_as_of_date_frequencies - how much time between rows for a single entity in a training matrix
- max_training_histories - the maximum amount of history for each entity to train on (early matrices may contain less than this time if it goes past label/feature start times)
- training_label_timespans - how much time is covered by training labels (e.g., outcomes in the next 1 year? 3 days? 2 months?)
- test_as_of_date_frequencies - how much time between rows for a single entity in a test matrix
- test_durations - how far into the future should a model be used to make predictions (in the typical case of wanting a single prediction set immediately after model training, this should be set to 0 days)
- test_label_timespans - how much time is covered by test predictions (e.g., outcomes in the next 1 year? 3 days? 2 months?)

Here's an example:

```
from timechop.timechop import Timechop

chopper = Timechop(
	feature_start_time=datetime.datetime(1990, 1, 1, 0, 0), 
    feature_end_time=datetime.datetime(2010, 1, 16, 0, 0),
    label_start_time=datetime.datetime(2010, 1, 1, 0, 0),
    label_end_time=datetime.datetime(2010, 1, 16, 0, 0),
    model_update_frequency='5 days',
    training_as_of_date_frequencies=['1 days'],
    max_training_histories=['5 days'],
    training_label_timespans=['1 day'],
    test_as_of_date_frequencies=['3 days'],
    test_durations=['5 days'],
    test_label_timespans=['3 days']
)
result = chopper.chop_time()
print(result)
```

```
[
    {
        'label_end_time': datetime.datetime(2010, 1, 16, 0, 0),
        'test_matrices': [{
            'test_data_frequency': '3 days',
            'as_of_times': [
                datetime.datetime(2010, 1, 4, 0, 0),
                datetime.datetime(2010, 1, 7, 0, 0)
            ],
            'matrix_end_time': datetime.datetime(2010, 1, 9, 0, 0),
            'matrix_start_time': datetime.datetime(2010, 1, 4, 0, 0),
            'test_span': '5 days',
            'test_prediction_span': '1 day'
        }],
        'train_matrix': {
            'training_data_frequency': '1 days',
            'matrix_start_time': datetime.datetime(2010, 1, 1, 0, 0),
            'max_training_history': '5 days',
            'as_of_times': [datetime.datetime(2010, 1, 1, 0, 0)],
            'matrix_end_time': datetime.datetime(2010, 1, 4, 0, 0),
            'training_prediction_span': '3 days'
        },
        'feature_end_time': datetime.datetime(2010, 1, 16, 0, 0),
        'feature_start_time': datetime.datetime(1990, 1, 1, 0, 0),
        'label_start_time': datetime.datetime(2010, 1, 1, 0, 0)
    },
    {
        'label_end_time': datetime.datetime(2010, 1, 16, 0, 0),
        'test_matrices': [{
            'test_data_frequency': '3 days',
            'as_of_times': [
                datetime.datetime(2010, 1, 9, 0, 0),
                datetime.datetime(2010, 1, 12, 0, 0)
            ],
            'matrix_end_time': datetime.datetime(2010, 1, 14, 0, 0),
            'matrix_start_time': datetime.datetime(2010, 1, 9, 0, 0),
            'test_span': '5 days',
            'test_prediction_span': '1 day'
        }],
        'train_matrix': {
            'training_data_frequency': '1 days',
            'matrix_start_time': datetime.datetime(2010, 1, 2, 0, 0),
            'max_training_history': '5 days',
            'as_of_times': [
                datetime.datetime(2010, 1, 2, 0, 0),
                datetime.datetime(2010, 1, 3, 0, 0),
                datetime.datetime(2010, 1, 4, 0, 0),
                datetime.datetime(2010, 1, 5, 0, 0),
                datetime.datetime(2010, 1, 6, 0, 0)
            ],
            'matrix_end_time': datetime.datetime(2010, 1, 9, 0, 0),
            'training_prediction_span': '3 days'
        },
        'feature_end_time': datetime.datetime(2010, 1, 16, 0, 0),
        'feature_start_time': datetime.datetime(1990, 1, 1, 0, 0),
        'label_start_time': datetime.datetime(2010, 1, 1, 0, 0)
    }
]
```

The output of Timechop works as input to the [architect.Planner](https://github.com/dssg/architect/blob/master/architect/planner.py).
