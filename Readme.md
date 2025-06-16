# MLflow on AWS Example

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![MLflow](https://img.shields.io/badge/MLflow-2.0+-green.svg)](https://mlflow.org/)
[![AWS](https://img.shields.io/badge/AWS-S3%20%7C%20EC2-orange.svg)](https://aws.amazon.com/)

This project demonstrates how to use [MLflow](https://mlflow.org/) for experiment tracking and model management with remote tracking and artifact storage on AWS. The example uses the Wine Quality dataset to train an ElasticNet regression model with centralized experiment tracking.

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Local Dev     │    │   EC2 Instance  │    │   S3 Bucket     │
│                 │    │                 │    │                 │
│  ┌───────────┐  │    │  ┌───────────┐  │    │  ┌───────────┐  │
│  │  app.py   │  │───▶│  │  MLflow   │  │───▶│  │ Artifacts │  │
│  │           │  │    │  │  Server   │  │    │  │  Storage  │  │
│  └───────────┘  │    │  └───────────┘  │    │  └───────────┘  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📁 Project Structure

```
DSProject/
├── .env                 # Environment variables (not tracked)
├── .gitignore          # Git ignore rules
├── app.py              # Main training script
├── README.md           # This file
├── requirements.txt    # Python dependencies
└── mlruns/            # Local MLflow runs (not tracked)
```

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- AWS Account with CLI configured
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/mlflow-aws-example.git
cd mlflow-aws-example
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Set Up Environment Variables

Create a `.env` file:

```bash
# Replace with your actual EC2 public DNS
MLFLOW_TRACKING_URI=http://your-ec2-public-dns:5000/
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_DEFAULT_REGION=your_region
```

### 4. Run the Training Script

```bash
# Run with default parameters
python app.py

# Run with custom hyperparameters
python app.py 0.5 0.5
```

## ⚙️ AWS Infrastructure Setup

### Step 1: IAM User Setup

1. Login to AWS Console
2. Create IAM user with `AdministratorAccess` policy
3. Download access keys

### Step 2: Configure AWS CLI

```bash
aws configure
# Enter your Access Key ID, Secret Access Key, and Region
```

### Step 3: Create S3 Bucket

```bash
# Replace 'your-unique-bucket-name' with your desired bucket name
aws s3 mb s3://your-unique-bucket-name
```

### Step 4: Launch EC2 Instance

1. Launch Ubuntu instance (t2.micro for testing)
2. Configure Security Group:
   - Allow SSH (port 22) from your IP
   - Allow Custom TCP (port 5000) from anywhere (0.0.0.0/0)

### Step 5: Set Up MLflow Server on EC2

Connect to your EC2 instance and run:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python and pip
sudo apt install python3-pip python3-venv -y

# Create virtual environment
python3 -m venv mlflow-env
source mlflow-env/bin/activate

# Install required packages
pip install mlflow boto3 awscli

# Configure AWS credentials
aws configure

# Start MLflow server (replace with your S3 bucket name)
mlflow server \
    -h 0.0.0.0 \
    -p 5000 \
    --default-artifact-root s3://your-unique-bucket-name/mlflow-artifacts \
    --backend-store-uri sqlite:///mlflow.db
```

### Step 6: Access MLflow UI

Open your browser and navigate to:
```
http://your-ec2-public-dns:5000
```

## 📊 Features

- **Remote Experiment Tracking**: All experiments logged to centralized MLflow server
- **Artifact Storage**: Models and artifacts stored in S3
- **Hyperparameter Tuning**: Easy parameter adjustment via command line
- **Metrics Logging**: Automatic logging of RMSE, MAE, and R² scores
- **Model Versioning**: Automatic model versioning and registration

## 🔧 Configuration

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `MLFLOW_TRACKING_URI` | MLflow server URL | `http://ec2-xxx.amazonaws.com:5000/` |
| `AWS_ACCESS_KEY_ID` | AWS access key | `AKIA...` |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | `xyz...` |
| `AWS_DEFAULT_REGION` | AWS region | `us-east-1` |

### Model Parameters

| Parameter | Description | Default | Range |
|-----------|-------------|---------|-------|
| `alpha` | Regularization strength | `0.5` | `0.0-1.0` |
| `l1_ratio` | ElasticNet mixing parameter | `0.5` | `0.0-1.0` |

## 📈 Usage Examples

### Basic Training

```bash
python app.py
```

### Custom Hyperparameters

```bash
# High regularization
python app.py 1.0 0.8

# Low regularization  
python app.py 0.1 0.2

# Ridge regression (l1_ratio=0)
python app.py 0.5 0.0

# Lasso regression (l1_ratio=1)
python app.py 0.5 1.0
```

### Batch Experiments

```bash
# Run multiple experiments with different parameters
for alpha in 0.1 0.5 1.0; do
    for l1_ratio in 0.0 0.5 1.0; do
        python app.py $alpha $l1_ratio
    done
done
```

## 🗂️ Dataset

This project uses the **Wine Quality Dataset** from the UCI Machine Learning Repository:
- **Source**: [Wine Quality Dataset](https://archive.ics.uci.edu/ml/datasets/wine+quality)
- **Samples**: ~4,898 wine samples
- **Features**: 11 physicochemical properties
- **Target**: Wine quality score (0-10)

## 🛠️ Development

### Local Development Setup

```bash
# Clone repository
git clone https://github.com/yourusername/mlflow-aws-example.git
cd mlflow-aws-example

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up pre-commit hooks (optional)
pip install pre-commit
pre-commit install
```

### Adding New Models

To add new models to the pipeline:

1. Import your model in `app.py`
2. Add model-specific parameters
3. Update the training loop
4. Log model-specific metrics

## 🔒 Security Best Practices

- ✅ Use IAM roles instead of access keys when possible
- ✅ Restrict S3 bucket access with proper policies
- ✅ Use VPC and security groups to limit EC2 access
- ✅ Never commit `.env` files or AWS credentials
- ✅ Regularly rotate access keys
- ⚠️ The current setup allows port 5000 from anywhere - restrict this in production

## 🚨 Troubleshooting

### Common Issues

**Connection Refused Error**
```bash
# Check if MLflow server is running
curl http://your-ec2-public-dns:5000

# Restart MLflow server on EC2
mlflow server -h 0.0.0.0 -p 5000 --default-artifact-root s3://your-bucket
```

**S3 Permission Denied**
```bash
# Verify AWS credentials
aws sts get-caller-identity

# Check S3 bucket permissions
aws s3 ls s3://your-bucket
```

**Module Import Errors**
```bash
# Ensure all dependencies are installed
pip install -r requirements.txt
```

## 📝 TODO

- [ ] Add Docker support
- [ ] Implement model serving endpoint
- [ ] Add automated testing
- [ ] Create Terraform templates for infrastructure
- [ ] Add monitoring and alerting
- [ ] Implement CI/CD pipeline

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [MLflow](https://mlflow.org/) for the excellent experiment tracking platform
- [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/) for the Wine Quality dataset
- AWS for cloud infrastructure services

## 📞 Support

If you encounter any issues or have questions:

1. Check the [Troubleshooting](#-troubleshooting) section
2. Search existing [Issues](https://github.com/yourusername/mlflow-aws-example/issues)
3. Create a new issue with detailed information

---

**⭐ Star this repository if you find it helpful!**
