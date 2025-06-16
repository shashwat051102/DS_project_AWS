# MLflow on AWS Example

This project demonstrates how to use [MLflow](https://mlflow.org/) for experiment tracking and model management, with remote tracking and artifact storage on AWS (S3 and EC2). The example uses the Wine Quality dataset and trains an ElasticNet regression model.

---

## Project Structure

```
DSProject/
├── .env
├── .gitignore
├── app.py
├── Readme.md
├── requirements.txt
└── mlruns/
```

- **app.py**: Main script for training and MLflow logging.
- **requirements.txt**: Python dependencies.
- **.env**: Environment variables (e.g., `MLFLOW_TRACKING_URI`).
- **mlruns/**: Local MLflow runs (ignored by git).

---

## MLflow on AWS Setup

1. **Login to AWS console.**
2. **Create IAM user** with `AdministratorAccess`.
3. **Configure AWS CLI** on your machine:
   ```bash
   aws configure
   ```
4. **Create an S3 bucket** (e.g., `mlflowtracking1`).
5. **Launch an EC2 instance** (Ubuntu recommended) and open port 5000 in the security group.

### On the EC2 Machine

```bash
sudo apt update
sudo apt install python3-pip
sudo apt install pipenv
sudo apt install virtualenv

mkdir mlflow
cd mlflow

pipenv install mlflow
pipenv install awscli
pipenv install boto3
pipenv shell

# Set AWS credentials
aws configure

# Start MLflow server
mlflow server -h 0.0.0.0 --default-artifact-root s3://mlflowtracking1
```

- Open the EC2 Public IPv4 DNS on port 5000 in your browser to access the MLflow UI.

---

## Local Setup

1. **Clone this repository** and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. **Set the MLflow tracking URI** in your terminal or `.env` file:
   ```bash
   export MLFLOW_TRACKING_URI=http://ec2-56-228-19-151.eu-north-1.compute.amazonaws.com:5000/
   ```

   Or add to your `.env`:
   ```
   MLFLOW_TRACKING_URI=http://ec2-56-228-19-151.eu-north-1.compute.amazonaws.com:5000/
   ```

---

## Running the Project

Run the training script with optional hyperparameters:
```bash
python app.py [alpha] [l1_ratio]
```
Example:
```bash
python app.py 0.5 0.5
```

- The script logs parameters (`alpha`, `l1_ratio`) and metrics (RMSE, MAE, R2) to the remote MLflow server.

---

## Notes

- The `.env` file and `mlruns/` directory are excluded from git.
- The script downloads the Wine Quality dataset from the internet.
- Make sure your AWS credentials are set up on both EC2 and your local machine.

---

## References

- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)
-
