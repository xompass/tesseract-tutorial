# tesseract-tutorial
Video Tutorial : https://www.youtube.com/watch?v=TpD76k2HYms

### Instalacion

[Instalacion tesseract](https://tesseract-ocr.github.io/tessdoc/Compiling-%E2%80%93-GitInstallation.html). Se debe compilar con las herramientas de entrenamiento.

Se puede instalar mediante un package manager siguiendo los siguientes pasos.
```bash
#Instalacion Tesseract
apt-get install tesseract-ocr
apt-get install tesseract-ocr-all
apt-get install imagemagick
```

```bash
#Instalacion librerias adicionales para entrenamiento
sudo apt-get install libicu-dev
sudo apt-get install libpango1.0-dev
sudo apt-get install libcairo2-dev
```

Luego, en el directorio de Tesseract
```bash
make
make training
sudo make training-install
```

### Entrenamiento

#### Generacion de imagenes
En la carpeta fonts, debe estar el font base sobre el cual se requiere entrenar.
Luego, copiar language-specific.sh  tesstrain.sh  tesstrain_utils.sh desde la carpeta de instalacion de tesseract. Usualmente el path es /usr/share/tesseract-ocr

Luego, ejecutar el siguiente comando
```bash
tesstrain.sh --fonts_dir fonts \
	--fontlist 'AgencyFB Bold' \
	--lang eng \
	--linedata_only \
	--langdata_dir langdata_lstm \
	--tessdata_dir tesseract/tessdata \
	--save_box_tiff \
	--maxpages 10 \
	--output_dir train
```
fontlist debe ser la fuente base sobre la cual se quiere entrenar.
Max pages se recomienda 200 para que genere una cantidad generosa de caracteres para entrenar.

Esto generará 200 páginas de texto y sus bbox correspondientes para el entrenamiento de tesseract. Luego, del repositorio de [Tesseract_best](https://github.com/tesseract-ocr/tessdata_best), descargar los mejores pesos para el lenguaje que se este entrenando, en este caso eng. 

Luego, para entrenar, correr  los siguientes comandos
```bash
rm -rf output/*
OMP_THREAD_LIMIT=16 lstmtraining \
	--continue_from eng.lstm \
	--model_output output/pubg \
	--traineddata tesseract/tessdata/eng.traineddata \
	--train_listfile train/eng.training_files.txt \
	--max_iterations 400
```

Esto generará pesos dentro de la carpeta output. 

Finalmente, hay que unirlos con los pesos originales para obtener el modelo nuevo, esto se hace corriendo el siguiente comando
```bash
lstmtraining --stop_training \
	--continue_from output/pubg_checkpoint \
	--traineddata tesseract/tessdata/eng.traineddata \
	--model_output output/pubg.traineddata

```