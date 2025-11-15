# Tutorial 10: AWS Observability Integration

## 📖 Overview

**What You'll Learn:**
- AWS CloudWatch integration
- AWS X-Ray distributed tracing
- AWS Bedrock monitoring
- Production deployment patterns
- Cost optimization on AWS

**Prerequisites:** [Tutorial 8: Monitoring Metrics](08_monitoring_metrics.md)

**Time to Complete:** 35 minutes | **Difficulty:** ⭐⭐⭐ Advanced

---

## ☁️ AWS CloudWatch

### Setting Up CloudWatch Logs

**For agent logs:**
```python
import boto3
import logging

# Create CloudWatch logs client
logs_client = boto3.client('logs')

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger('agent')

# Log agent events
logger.info("Agent started", extra={
    "user_id": "user_123",
    "session_id": "sess_456"
})
```

### CloudWatch Metrics

**Track custom metrics:**
```python
cloudwatch = boto3.client('cloudwatch')

# Put custom metric
cloudwatch.put_metric_data(
    Namespace='AgentMetrics',
    MetricData=[
        {
            'MetricName': 'RequestLatency',
            'Value': 2.3,
            'Unit': 'Seconds',
            'Timestamp': datetime.now()
        }
    ]
)
```

---

## 🔍 AWS X-Ray

### Setting Up X-Ray Tracing

**Install SDK:**
```bash
pip install aws-xray-sdk
```

**Instrument your code:**
```python
from aws_xray_sdk.core import xray_recorder

# Start segment
with xray_recorder.capture('agent_execution'):
    # Your agent code
    result = agent.invoke(...)
    
    # Add metadata
    xray_recorder.current_segment().put_metadata(
        'user_id', 'user_123'
    )
```

---

## 📊 AWS Bedrock Monitoring

### Tracking Bedrock API Calls

**CloudWatch automatically tracks:**
- API call count
- Token usage
- Latency
- Errors

**Access metrics:**
1. Go to CloudWatch Console
2. Navigate to Metrics
3. Select "AWS/Bedrock"
4. View metrics dashboard

### Cost Tracking

**Monitor costs:**
- Input tokens × price
- Output tokens × price
- Total requests
- Cost by model

**Set budget alerts:**
```python
# Create budget alert
budgets = boto3.client('budgets')

budgets.create_budget(
    AccountId='123456789012',
    Budget={
        'BudgetName': 'Agent-Monthly-Budget',
        'BudgetLimit': {
            'Amount': '1000',
            'Unit': 'USD'
        },
        'TimeUnit': 'MONTHLY'
    }
)
```

---

## 🎯 Production Deployment Patterns

### Pattern 1: Lambda + EventBridge
```
EventBridge → Lambda → Bedrock
                ↓
           CloudWatch Logs
```

### Pattern 2: ECS + Application Load Balancer
```
ALB → ECS Task → Bedrock
         ↓
    CloudWatch + X-Ray
```

### Pattern 3: SageMaker Endpoint
```
API Gateway → SageMaker Endpoint → Bedrock
                      ↓
              CloudWatch Monitoring
```

---

## 🎓 Key Takeaways

**AWS Observability Tools:**
- CloudWatch (logs + metrics)
- X-Ray (distributed tracing)
- Bedrock built-in monitoring
- Cost tracking and budgets

**Best Practices:**
- Log everything to CloudWatch
- Use X-Ray for complex workflows
- Monitor Bedrock usage
- Set up cost alerts

**Next Steps:**
- [Tutorial 11: Production Practices](11_production_practices.md)
- [Tutorial 12: Hands-On Exercises](12_hands_on_exercises.md)

---

**🎉 Well done!** You can now use AWS observability tools! Continue to [Tutorial 11: Production Practices](11_production_practices.md)!
