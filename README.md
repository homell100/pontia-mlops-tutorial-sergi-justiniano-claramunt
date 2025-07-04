# Proyecto de integración de modelo de ML
El proyecto entrena un modelo de RandomForest con datos tabulares, con la capacidad de desplegarlo en local o en un contenedor de Azure MLFlow

## Requisitos previos

Para conseguir entrenar el modelo en un entorno local es necesario disponer del dataset 'adult-income' en el directorio `data/`, de forma que los datos de entrenamiento sean `data/adult.data`y los de test `data/adult.test`. El dataset se puede encontrar en la siguiente url `https://archive.ics.uci.edu/dataset/2/adult`. 

Para conseguir entrenar el modelo utilizando la funcionalidad de un contenedor de Azure usando MLFlow es necesario definir los siguientes valores en GitHub actions:
- Secretos: `AZURE_CREDENTIALS`, `ACR_NAME`, `ACR_USERNAME`, `ACR_PASSWORD`, `AZURE_RESOURCE_GROUP`, `AZURE_STORAGE_CONNECTION_STRING`
- Variables: `EXPERIMENT_NAME`, `MLFLOW_URL`, `MODEL_NAME`

## Ejecución en entorno local

El modelo se puede entrenar en local instalando las dependencias

```
pip install -r requirements.txt
```

entrenando el modelo

```
python src/main.py
```

## Despliege automático 

El despliegue y entrenamiento se realiza automáticamente al hacer push a la rama main o ejecutando manualmente los workflows desde GitHub Actions.

## Como funciona la integración continua?

- El workflow definido en `integration.yml` se ejecuta al realizar cada Pull Request, permitiendo instalar dependencias y ejecutando los test definidos en `unit_test/`, permitiendo ver los resultados en los comentarios del mismo Pull Request, impidiendo esta si los test fallan.

- El workflow definido en `build.yml` se encarga instalar dependecias, descargar datos de entrenamiento, entrenar el modelo, ejecutar los tests y registrar el modelo en MLFlow.

- El workflow definido en `deploy.yml` construye una imagen Docker del modelo, la publica en Azure Container Registry, construye un contenedor de ella y habililita un API endpoint para poder consultarlo.
- 
## Como consultar el modelo a través de la API?

Una vez el workflow deploy haya fianlizado, el modelo puede ser consultado a través de el endpoint:

`http:/model-api-<IMAGE_NAME>-<RUN_ID>.eastus.azurecontainer.io:8080\predict`

donde IMAGE_NAME se ha definido como variable en 'deploy.yml`y RUN_ID es un id autogenerado por GitHub que se puede consultar en los logs del workflow deploy.

### Ejemplo:

![image](https://github.com/user-attachments/assets/cf972623-786e-4ac3-8e00-6663b042e7d2)

## Dificultades durante el proyecto:

Principalmente la adapatición del codigo a posibles canvios en las variables y el uso de secretos y variables en GitHub, las cuales permiten tener aquellos parámetros bajo control una vez se ha definido todo el proyecto correctamente, pero que han generado diversos errores mientras se desarrollaba.

