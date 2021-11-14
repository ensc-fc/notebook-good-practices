# Notebook good practices

Demo project for applying good practices to Jupyter notebooks.

[Initial code](v0-initial-code/) is lacking in many ways. A series of refactoring steps can turn it into a well organized and reusable project.

## Refactoring steps

1. Improve notebook

   - Add Markdown cells to describe the notebook and indicate its main parts.
   - Change code to be more pythonic.
   - Create functions to prevent code duplication.

1. Manage dependencies

   - Install and init [Poetry](https://python-poetry.org).
   - Configure Poetry to create the virtual environment in the project folder.

     ```bash
     poetry config virtualenvs.in-project true
     ```

   - Configure dependencies according to the following versions:

     ```toml
     [tool.poetry.dependencies]
     python = "^3.7"
     jupyter = "^1.0.0"
     matplotlib = "^3.3.2"
     sklearn = "^0.0"
     pandas = "^1.1.3"
     tensorflow = "^2.3.1"
     seaborn = "^0.11.0"
     black = "^21.10b0"
     ```

   - Install dependencies and test the notebook.

1. Organize notebooks

   - Split the notebook into four separate parts for loading and transforming data, visualizing data, training models and visualizing results. Transformed data should be serialized as JSON files like so:

     ```python
     training_data = {
        "x_train": x_train.tolist(),
        "x_test": x_test.tolist(),
        "y_train": y_train.tolist(),
        "y_test": y_test.tolist(),
        "n_features": n_features,
        "n_classes": n_classes
     }
     with open("iris_training_data.json", "w+") as f:
        json.dump(training_data, f, ensure_ascii=False, indent=4)
        # This file can be opened with json.load()

     # Same for visualization data
     ```

   - More raw and processed data into a dedicated `data/` subdirectory.
   - Save models after training into a dedicated `models/` subdirectory.

1. Externalize code

   - Move code for data preprocessing, model creation and training, and visualizations to functions in Python files `data_transformation.py`, `training.py` and `visualizations.py`. These files must be located in a dedicated `src/` subfolder also containing an empty `__init__.py` file.
   - Import and use this code in the notebooks like so:

     ```python
     import sys
     sys.path.append("..")
     from src.data_transformation import one_hot_encode, get_data_and_names, scale, split
     ```

1. Add unit tests

   - Add [pytest](https://docs.pytest.org) to development dependencies and install it.
   - Create unit tests for data preprocessing functions in a dedicated `tests/` subfolder. Tests should be run using the following command.

     ```bash
     python - m pytest tests/
     ```

1. Create a notebook workflow

   - Add [papermill](https://papermill.readthedocs.io) to dependencies and install it.
   - Update the training notebook to parameterize the number of models created, through a Papermill parameter named `n_models`.
   - Create a workflow notebook which runs the four other notebooks in correct order, using Papermill like so:

     ```python
     import papermill as pm
     import time
     from pathlib import Path
     import os

     timestamp = time.time()
     base_folder = f"./papermill_executions/{timestamp}"
     Path(base_folder).mkdir(parents=True, exist_ok=True)

     # Same for other notebooks (excepting parameter)
     notebook_name = "3-train-models.ipynb"
     pm.execute_notebook(
         notebook_name,
         os.path.join(base_folder, notebook_name),
         parameters=dict(n_models=4)
     );
     ```
