# IA_Embarque
Projet collaboratif 

## Probléme

Lors de l'exportation du model des valeurs non numerique se glisse dans les données de validations et bloque l'analyse de Cub_IDE:\
Message de Cube_IDE:\
```
Analyzing model 
C:/Users/hugoc/STM32Cube/Repository/Packs/STMicroelectronics/X-CUBE-AI/10.0.0/Utilities/windows/stedgeai.exe analyze --target stm32l4 --name my_model_v1 -m C:/Users/hugoc/OneDrive/Documents/Hugo/ISMIN/Cours/S8/Ia_Data/IA_embarque/Github/IA_Embarque/COLAB/Model_V1.h5 --compression none --verbosity 1 --workspace C:/Users/hugoc/AppData/Local/Temp/mxAI_workspace88344567030008779692142564994152 --output C:/Users/hugoc/.stm32cubemx/my_model_v1_output 
ST Edge AI Core v2.0.0-20049 
E010(InvalidModelError): Non-numerical or not-string elements in dense_60_dense's parameter weights
```