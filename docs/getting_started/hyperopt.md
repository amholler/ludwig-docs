After training our first model and using it to predict new data with reasonable accuracy, how can we make the model
better?

Ludwig can perform hyperparameter optimization by simply adding `hyperopt` to the Ludwig config.

```yaml title="rotten_tomatoes.yaml"
input_features:
    - name: genres
      type: set
    - name: content_rating
      type: category
    - name: top_critic
      type: binary
    - name: runtime
      type: number
    - name: review_content
      type: text
      encoder: embed
output_features:
    - name: recommended
      type: binary
hyperopt:
  goal: maximize
  output_feature: recommended
  metric: accuracy
  split: validation
  parameters:
    training.learning_rate:
      type: float
      low: 0.0001
      high: 0.1
      steps: 4
      scale: log
    training.optimizer.type:
      type: category
      values: [sgd, adam, adagrad]
    review_content.embedding_size:
      type: category
      values: [128, 256]
  sampler:
    type: random
  executor:
    type: serial
```

In this example we have specified a basic hyperopt config with the following specifications:

* We have set the `goal` to maximize the **accuracy** metric on the **validation** split
* The parameters we are optimizing are the **learning rate**, the **optimizer type**, and the **level** of text representation to use.
  * When optimizing **learning rate** we are testing *four* values on a *log* scale between 0.0001 and 0.1 ([0.0001, 0.001, 0.01, 0.1]).
  * When optimizing the **optimizer type**, we are testing the *sgd*, *adam*, and *adagrad* optimizers
  * When optimizing the **level** of text representation to use, we are testing representations at the *word* level and the *char* level
* We set the hyperopt `sampler` to use the random sampler. This selects 10 random hyperparameter combinations from the search space by default.
  * Ludwig supports advanced hyperparameter sampling algorithms like Bayesian optimization and genetical algorithms. See [this guide](../../configuration/hyperparameter_optimization#sampler) for details.
* We set the hyperopt `executor` to use the serial executor which performs the optimization locally in a serial manner.

The hyperparameter optimization strategy is run using the ludwig hyperopt command:

=== "CLI"

    ```sh
    ludwig hyperopt --config rotten_tomatoes.yaml --dataset rotten_tomatoes.csv
    ```

=== "Python"

    ```python
    from ludwig.hyperopt.run import hyperopt
    import pandas

    df = pandas.read_csv('rotten_tomatoes.csv')
    results = hyperopt(config='rotten_tomatoes.yaml', dataset=df)
    ```

=== "Docker CLI"

    ```sh
    docker run -t -i --mount type=bind,source={absolute/path/to/rotten_tomatoes_data},target=/rotten_tomatoes_data ludwigai/ludwig hyperopt --config /rotten_tomatoes_data/rotten_tomatoes.yaml --dataset /rotten_tomatoes_data/rotten_tomatoes.csv
    ```

Every parameter within the config can be tuned using hyperopt. Refer to the full [hyperopt guide](../../configuration/hyperparameter_optimization) to learn more.
