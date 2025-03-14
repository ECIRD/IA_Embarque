# IA_Embarqué  
**Projet collaboratif**  

## Arborescence  

Ce GitHub est organisé de la manière suivante :  
- Un répertoire **COLAB** contenant :  
    - Le Jupyter Notebook au format **.ipynb**  
    - Le modèle MLP exporté au format **.h5**  
    - Les données d'entraînement au format **.csv**  
    - Les données de validation au format **.npy**  
- Un répertoire **UART** contenant :  
    - Un script Python au format **.py** permettant de lire les données reçues via un port UART  
- Un répertoire **CUBE_IDE** contenant :  
    - Un répertoire **.metadata** pour gérer les logs du workspace  
    - Un répertoire **IA_Emb_Pablo_Hugo** contenant le projet CubeIDE  

## Problème  

Lors de l'exportation du modèle, des valeurs non numériques se glissent dans les données de validation, ce qui bloque l'analyse par CubeIDE.  

### Message d'erreur de CubeIDE :  
```
Analyzing model 
C:/Users/hugoc/STM32Cube/Repository/Packs/STMicroelectronics/X-CUBE-AI/10.0.0/Utilities/windows/stedgeai.exe analyze --target stm32l4 --name my_model_v1 -m C:/Users/hugoc/OneDrive/Documents/Hugo/ISMIN/Cours/S8/Ia_Data/IA_embarque/Github/IA_Embarque/COLAB/Model_V1.h5 --compression none --verbosity 1 --workspace C:/Users/hugoc/AppData/Local/Temp/mxAI_workspace88344567030008779692142564994152 --output C:/Users/hugoc/.stm32cubemx/my_model_v1_output 
ST Edge AI Core v2.0.0-20049 
E010(InvalidModelError): Non-numerical or not-string elements in dense_60_dense's parameter weights
```

---
