# Streaming data en tiempo real: Análisis predictivo mediante Regresión Lineal

![Banner](docs/assets/images/banner_delgado3.jpg)

### Código Python:

### 1. Importar bibliotecas
```
import json
from kafka import KafkaConsumer
from river import linear_model, preprocessing, metrics
```
### 2. Kafka consumer para datos en tiempo real
```
consumer = KafkaConsumer(
    'realtime-data',
    bootstrap_servers='localhost:9092',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)
```
### 3. River: modelo de regresión online con escalamiento de variables independientes (features)
```
model = preprocessing.StandardScaler() | linear_model.LinearRegression()
mae = metrics.MAE()

for message in consumer:
    data = message.value
    # Datos de entrada de ejemplo: {'f1': 1.0, 'f2': 2.5, 'target': 10.2}
    y = data.pop('target')
    X = data

    # Predecir y actualizar
    # Cuando `predict_one` es invocado antes de que el modelo haya recibido datos, `y_pred` puede recibir el valor `None`:
    y_pred = model.predict_one(X) or 0.0 
    print(f"Predicted: {y_pred:.2f} | Actual: {y}")
    model.learn_one(X, y)
    mae.update(y, y_pred)
    print(f"Current MAE: {mae.get():.4f}")
```
