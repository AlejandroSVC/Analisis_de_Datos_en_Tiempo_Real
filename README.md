# Tiempo real: Análisis predictivo mediante Regresión Lineal

![Banner](docs/assets/images/banner.jpg)

### Python code:

### 1. Import libraries
```
import json
from kafka import KafkaConsumer
from river import linear_model, preprocessing, metrics
```
### 2. Kafka consumer for real-time data
```
consumer = KafkaConsumer(
    'realtime-data',
    bootstrap_servers='localhost:9092',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)
```
### 3. River: Online regression model with feature scaling
```
model = preprocessing.StandardScaler() | linear_model.LinearRegression()
mae = metrics.MAE()

for message in consumer:
    data = message.value
    # Example input: {'f1': 1.0, 'f2': 2.5, 'target': 10.2}
    y = data.pop('target')
    X = data

    # Predict and update
    # When `predict_one` is called before the model has seen any data, `y_pred` can be `None`:
    y_pred = model.predict_one(X) or 0.0 
    print(f"Predicted: {y_pred:.2f} | Actual: {y}")
    model.learn_one(X, y)
    mae.update(y, y_pred)
    print(f"Current MAE: {mae.get():.4f}")
```
