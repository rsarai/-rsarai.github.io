---
layout: post
title: The Stupid Mistake I Did The Most During My Masters
categories: [research]
excerpt: In my quest to populate this blog with relevant content, I'm willing to share the most embarrassing and rookie mistakes I've made (multiple times) during my two-year master's program. My hope is to help some other rookie like me or entertain someone more experienced.
last_modified_at:
---


1.  [The mistake](#org2c15c6c)
2.  [Sanity Check](#orgc8cdb58)
3.  [Different ways to make the same mistake](#org5d56c76)
    1.  [Case #1](#orgb7f5e86)
    2.  [Case #2](#org0eecb31)
    3.  [Case #3](#orgd7561cb)
4.  [Conclusion](#orge2d5aed)


In my quest to populate this blog with relevant content, I'm willing to share the most embarrassing and rookie mistakes I've made (multiple times) during my two-year master's program. My hope is to help some other rookie like me or entertain someone more experienced.

To give you context, I'm pursuing an MS in CS at the University of Pernambuco, I have no funding and an 8 hours job. My research focus is on the intersection of artificial intelligence and healthcare. I work with gait signals to detect early signals of neurodegenerative diseases and to improve evaluation/treatments of gait impairments with physiotherapy. If you are confused about what gait means, it means walking, automatic behavior that demands none or little cognition effort. You may also be confused about the utility of gait information, and I can assure you that it is quite useful. For example, many researchers point to gait as a unique identifiable factor which means that there are no two persons with the same gait, other studies use gait information to classify age and gender, predict conditions and propose treatments. At the time of writing, my focus is full-mode on writing the dissertation and finetuning the results I already have, therefore a good opportunity to look back to the months dedicated to research and development of the algorithms.

Disclaimer: I don't own a Ph.D. in machine learning (not even a master's degree), yet I have a big interest in the field and try to learn as much as I can about it. I'm always glad to hear any comments or feedback coming from you, so feel free to drop me an email.


<a id="org2c15c6c"></a>

# The mistake

As a first-timer researcher, I have made many mistakes the ones that come to mind quickly are not keeping organized notes about papers, not reading more papers, not having an organized way to save and compare experiments with different models, and not focusing on fundamentals first. However, the biggest mistake, that fooled my perception of progress, that made me lose time, and made me frustrated was:

    data leakage / overfitting / bias inclusion

It sounds silly and goofy, but I can't tell you the different ways I end up contaminating my models with so-called common practices, but wait, I can tell you.


<a id="orgc8cdb58"></a>

# Sanity Check

Before starting on the master's program I graduated in computer engineering. This means that I took classes on artificial intelligence and neural networks, therefore even before starting on the program if you asked me:

-   Why overfitting is bad?
-   How to avoid overfitting?
-   What would be the most wrong way to add bias into a model?

I would definitely know how to answer them. I would say that overfitting is when your model instead of learning to identify the patterns starts to decorate the behavior of the data decreasing its ability to generalize on new samples. To avoid this memorization, one solution is to split the data into training, validation, and test sets, you would only train your model with the training set, decide when to stop the training with the validation set, and then evaluate your model with the test set (👍🏾). This answer is not incorrect but is not complete either. Knowing these rules doesn't prevent you from making mistakes, in fact, it blinds you when you do.


<a id="org5d56c76"></a>

# Different ways to make the same mistake

A good metaphor for this is when you decide to build a vegetable garden in your house. You study, watch videos, read books, but still, after a month the vegetables or are not growing at all or they are being infested with plagues, maybe because of the soil, or fertilizer, or weather, the variables are almost infinite and things can go wrong in many ways. Machine learning algorithms are the same (with some sense of proportion, it still is a metaphor), even after studying and using known libraries you can still make mistakes. This means you are not a genius, you are human (😱), luckily you will learn from your mistakes. The next sections present some of the examples that happened to me, most of them were simplified to fit in this blog post.


<a id="orgb7f5e86"></a>

## Case #1

Check the following snippet. Let me give you 30 seconds to figure out what is going on&#x2026; In simple terms, I'm using an existing DataFrame, instantiating a standard scaler, normalizing my data, splitting the data into train/test sets, and passing it to a classifier.

Anything wrong here?

```python
    from sklearn.svm import SVC
    from sklearn.preprocessing import StandardScaler

    # df = dataframe with rows and columns

    sc = StandardScaler()
    df_normalized = sc.fit_transform(df)

    X = df_normalized.drop(['Class'], axis=1).values
    y = df_normalized['Class'].values
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.4, random_state=0
    )

    clf = SVC()
    clf.fit(X_train, y_train)
    y_test = clf.predict(X_test)
```
This would not be a problem if I was using another type of scaler (it would be just error-prone), however, in this case, the behavior of the standard scaler is the source of the data leakage. The scaler works by removing the mean and scaling to unit variance, this means that to do these operations the algorithm is considering the whole DataFrame including the test part which I'm not supposed to see until evaluation time. Even though unaware of it, I'm leaking output information to the training of my model. The solution is to move the database split to any point before the standardization.
```python
    from sklearn.svm import SVC
    from sklearn.preprocessing import StandardScaler

    # df = dataframe with rows and columns

    sc = StandardScaler()

    X = df.drop(['Class'], axis=1).values
    y = df['Class'].values
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.4, random_state=0
    ) # [THIS] moved up

    clf = SVC()

    X_train = sc.fit_transform(X_train)
    clf.fit(X_train, y_train)

    X_test = sc.fit_transform(X_test)
    y_test = clf.predict(X_test)
```

<a id="orga9407d2"></a>

### Life context & Consequences

This mistake was easy to catch. Through most of my research time I had a weekly sync with a friend and fellow student, we used the time to discuss research problems, plans, and implementation strategies. This was something she promptly spotted and things got fixed quickly. The consequences to my model metrics were none, the information was being leaked but after the fix, the results experienced no alteration.


<a id="org0eecb31"></a>

## Case #2

In this use case, I was exploring the use of neural networks to solve a classification problem. I decided to use a Stratified Cross-Validation with k=4 to evaluate the performance of the network defined in `build_network`. Can you spot the error below? Note that I completely omitted any type of preprocessing or standardization even though they are extremely important to the final performance of the model, the goal is to show that this is a different use case.
```python
    from sklearn.model_selection import StratifiedKFold

    def build_network():
        ann_model = keras.models.Sequential()
        ann_model.add(keras.layers.InputLayer(
            input_shape=[713])
        )
        ann_model.add(
            keras.layers.Dense(units=20, activation="tanh")
        )
        ann_model.add(
            keras.layers.Dense(1, activation='softmax')
        )
        ann_model.compile(
    	loss='binary_crossentropy',
    	optimizer=keras.optimizers.Adam(),
    	metrics=['accuracy']
        )
        return ann_model

    # df = dataframe with rows and columns

    X = df.drop(['Class'], axis=1).values
    y = df['Class'].values
    ann_model = build_network()

    kf = StratifiedKFold(
        shuffle=True, random_state=42, n_splits=4
    )
    for train_index, test_index in kf.split(X, y):
        X_train, X_test = X[train_index], X[test_index]
        y_train, y_test = y[train_index], y[test_index]

        ann_model.fit(
    	X_train,
    	y_train,
    	epochs=400,
    	verbose=False,
    	validation_split=0.3
        )
        _, train_acc = ann_model.evaluate(
            X_train, y_train, verbose=0
        )
        _, test_acc = ann_model.evaluate(
            X_test, y_test, verbose=0
        )
```
The root cause of this problem was uniquely between the chair and the keyboard (me 💁‍♀️). The idea behind k-fold cross-validation is to build different models by randomly partitioning the original data into k equal-sized subsamples and using K-1 subsets for training while the remaining one for testing. The total performance is achieved by averaging the results of the different classifiers. If you look closely at the code before you will see that the `build_network` function is being called one time, before the k-fold loop, this means that the training is happening with a single classifier multiple times with all the data. I'm not only leaking information to my model, I'm pointing a gun in her head forcing her to overfit. The fix is quite simple as you can see below.
```python
    from sklearn.model_selection import StratifiedKFold

    def build_network():
        ann_model = keras.models.Sequential()
        ann_model.add(keras.layers.InputLayer(
            input_shape=[713]
        ))
        ann_model.add(keras.layers.Dense(units=20, activation="tanh"))
        ann_model.add(keras.layers.Dense(1, activation='softmax'))
        ann_model.compile(
    	loss='binary_crossentropy',
    	optimizer=keras.optimizers.Adam(),
    	metrics=['accuracy']
        )
        return ann_model

    # df = dataframe with rows and columns
    X = df.drop(['Class'], axis=1).values
    y = df['Class'].values

    kf = StratifiedKFold(
        shuffle=True, random_state=42, n_splits=4
    )
    for train_index, test_index in kf.split(X, y):
        ann_model = build_network()    # [THIS] moved inside the loop
        X_train, X_test = X[train_index], X[test_index]
        y_train, y_test = y[train_index], y[test_index]

        ann_model.fit(
    	X_train,
    	y_train,
    	epochs=400,
    	verbose=False,
    	validation_split=0.3
        )
        _, train_acc = ann_model.evaluate(
            X_train, y_train, verbose=0
        )
        _, test_acc = ann_model.evaluate(
            X_test, y_test, verbose=0
        )
```

<a id="orgf47cb49"></a>

### Life context & Consequences

I must call this a distracted programmer mistake. In addition, I was distracted by this for a long time, results were always good, so good that I started to suspect. What gave it away was the individual metrics of each fold which presented an ascendant pattern (0.58, 0.69, 0.79, 0.83, 0.95). I'm embarrassed to say that I was blind to this mistake for 3 days. I blame my poor code organization (which is also ridiculous since I make a living from software development).


<a id="orgd7561cb"></a>

## Case #3

This use case is a bit different from the previous ones. I'm trying to decrease the dimensionality of my data before doing any type of classification. I picked a ranking algorithm that recently appeared on my research articles, called Relief, to do the task of ranking and selecting the best features.
```python
    from sklearn.preprocessing import StandardScaler
    from skrebate import ReliefF

    # df = dataframe with rows and columns

    sc = StandardScaler()
    X = df.drop(['Class'], axis=1).values
    y = df['Class'].values

    fs = ReliefF(
        n_features_to_select=2, n_neighbors=100, verbose=True
    )
    fs.fit(X, y)
    X_subset = fs.transform(X)

    X_train, X_test, y_train, y_test = train_test_split(
        X_subset, y, test_size=0.4, random_state=0
    )

    clf = SVC()
    X_train = sc.fit_transform(X_train)
    clf.fit(X_train, y_train)
    y_test = clf.predict(X_test)
```
I'm sure you already spotted the error, it builds on top of the previous cases. My feature selection algorithm is using the whole data, this means that the feature selection step used to extract relevant features is biased.
```python
    from sklearn.preprocessing import StandardScaler
    from skrebate import ReliefF
    # df = dataframe with rows and columns

    sc = StandardScaler()

    X = df.drop(['Class'], axis=1).values
    y = df['Class'].values
    X_train, X_test, y_train, y_test = train_test_split(
        X_subset, y, test_size=0.4, random_state=0
    )

    clf = SVC()
    X_train = sc.fit_transform(X_train)

    fs = ReliefF(
        n_features_to_select=2, n_neighbors=100, verbose=True
    )
    fs.fit(X_train, y_train) # [THIS] moved down
    X_train_subset = fs.transform(X_train)

    clf.fit(X_train_subset, y_train)

    X_test = sc.fit_transform(X_test)
    X_test_subset = fs.transform(X_test)
    y_test = clf.predict(X_test_subset)
```

<a id="org9856389"></a>

### Life context & Consequences

This use case had no consequences since I easily spotted it after writing the code. I dare to say that I have learned the lesson.


<a id="orge2d5aed"></a>

# Conclusion

In this post, you saw that even with a good understanding of the theory, practical situations often present challenges that may lead to mistakes in the application of simple concepts. I work every day as a software developer using Python, I also graduated in Computer Engineering from a good University, and I made some beginner mistakes when trying to apply the theory that I learned in class to practice. Our education system is not the best at preparing students to transfer learned topics from one context to another. In theory, I knew what to do, in practice, not so much. The takeaway is: when applying previous consolidated knowledge to new situations we must always check what we know and what is the main principle behind what we are trying to do. In this case, the principle behind all the silly mistakes is: the test set must be as far as possible from the model, your model should be unaware of its existence, except on the evaluation moment.

