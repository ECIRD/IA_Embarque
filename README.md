

# 🧠 IA Embarquée  
**Projet collaboratif de déploiement de modèle d’intelligence artificielle sur STM32**

---

## 👥 Collaborateurs

- Hugo CELARIE  
- Pablo COLIN  

---

## 📁 Arborescence du Dépôt

Ce dépôt GitHub est structuré comme suit :

- `COLAB/`  
  - Notebook Jupyter `TP_IA_EMBARQUEE.ipynb`  
  - Modèle MLP entraîné exporté au format `.h5`  
  - Données d'entraînement au format `.csv`  
  - Données de test/validation au format `.npy`  

- `UART/`  
  - Script Python `Serial_Uart.py` pour lire les données UART

- `CUBE_IDE/`  
  - `.metadata/` : fichiers internes du workspace CubeIDE  
  - `IA_Emb_Pablo_Hugo/` : projet CubeIDE contenant l’implémentation STM32  
  - `IA_Emb_Pablo_Hugo.zip` : archive compressée du projet  

- `Images/`  
  - Captures d’écran illustrant les différentes étapes de développement pour la rédaction du rapport final  

---

## 📘 Manuel d’Utilisation

Pour plus de détails sur l'intégration complète du modèle et sa mise en œuvre sur une carte STM32, veuillez vous référer au **Rapport IA Embarquée** ci-dessous.

---

### 🔬 Entraînement et Export du Modèle (via Google Colab)

1. **Chargement des données**  
   Assurez-vous de spécifier le chemin de vos données dans le notebook :

   ```python
   file_path = "/content/drive/MyDrive/Colab Notebooks/data/ai4i2020.csv"
   ```

2. **Exécution complète du notebook**  
   Lancez l’exécution de toutes les cellules du notebook pour prétraiter les données, entraîner le modèle et le sauvegarder.

3. **Export du modèle et des jeux de données**  
   À la fin du notebook, vous pouvez exporter les fichiers nécessaires à l’inférence embarquée :

   ```python
   np.save("X_test.npy", X_test)
   np.save("Y_test.npy", Y_test)
   model.save("Model_V1.h5")
   ```

---

### 🛠️ Implémentation sur Carte STM32 (via STM32CubeIDE)

Ce projet est conçu pour fonctionner avec une carte STM32 utilisant **STM32CubeIDE** et **X-CUBE-AI**.

#### Étapes :

1. Ouvrez le fichier `.ioc` via STM32CubeIDE.

2. Allez dans l’onglet **X-CUBE-AI**, importez le modèle `Model_V1.h5` en mode **Keras**, puis testez l’inférence directement avec les données `X_test.npy`.

3. Configurez l’interface UART2 avec un débit de **115200 bit/s**.

4. Dans le fichier `/X-CUBE-AI/App/app_x-cube-ai.c`, ajoutez/modifiez les définitions suivantes :

   ```c
   /* USER CODE BEGIN includes */
   extern UART_HandleTypeDef huart2;

   #define BYTES_IN_FLOATS 5*4
   #define TIMEOUT 1000
   #define SYNCHRONISATION 0xAB
   #define ACKNOWLEDGE 0xCD
   #define CLASS_NUMBER 5

   void synchronize_UART(void);
   /* USER CODE END includes */
   ```

5. Dans le fichier `CUBE_IDE/IA_Embedded_Vf/Core/Src/main.c` modifier les initialisation des fonction MX en enlevant l'initialisation de la fonction `MX_SDMMC1_SD_Init()` pour obtenir les initialisations suivantes:
```c
  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_FMC_Init();
  MX_I2C1_Init();
  MX_SAI1_Init();
  //MX_SDMMC1_SD_Init();
  MX_SPI2_Init();
  MX_USART2_UART_Init();
  MX_USART3_UART_Init();
  MX_USB_OTG_FS_PCD_Init();
  MX_X_CUBE_AI_Init();
```

6. Compilez et lancez le **mode Debug** dans CubeIDE pour déployer le firmware sur la carte.

---

### 🔄 Communication UART avec le script Python

Le script `Serial_Uart.py` permet de lire les sorties envoyées par la carte STM32 via UART.

#### Étapes :

1. Définissez le port série utilisé (automatique ou manuel). Par exemple, pour le port COM5 :

   ```python
   PORT = "COM5"
   ```

2. Chargez les données de validation :

   ```python
   X_test = np.load("../COLAB/X_test.npy")
   Y_test = np.load("../COLAB/Y_test.npy")
   ```

3. Lancez le script : les informations s'affichent dans le terminal, notamment :

   - `Iteration` : numéro d’itération
   - `Expected output` : sortie attendue
   - `Received output` : sortie prédite par la STM32
   - `Accuracy` : précision estimée du modèle en temps réel

---

# Rapport IA Embarquée

- Hugo CELARIE
- Pablo COLIN


L'objectif du projet est de developper un modéle d'IA faissant de la prédiction de maintenance puis d'implementer ce modele sur une carte **STM32R9**.

## Plan
1. **Analyse du probleme**
2. **Mise en place et analyse d'un modele d'IA**
3. **Implémentation sous cube IDE**
4. **Connexion UART**
5. **Analyse des resultats**

## 1. Analyse du probleme

Nous partons d'une base de données organisée de la façon suivante:

UDI|Product ID|Type|Air temperature [K]|Process temperature [K]|Rotational speed [rpm]|Torque [Nm]|Tool wear [min]|Machine failure|TWF|HDF|PWF|OSF|RNF|
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
*int*|*string*|*string*|*float*|*float*|*int*|*float*|*int*|*int*|*int*|*int*|*int*|*int*|*int*|

Où TWF, HDF, PWF, OSF et RNF sont des types d'erreur de production.

Notre but étant de prédire les maintenances, nous allons devoir prendre en compte les différents types de problèmes ainsi que les paramètres des machines au moment du problème et le type de produit pour essayer de trouver une corrélation.

## 2. Mise en place et analyse d'un modele d'IA

Une fois nos données importées, nous avons procédé à une première analyse du lot qu'il nous était fourni afin d'établir la bonne approche pour créer notre modèle et l'entrainer correctement.

![Distribution des pannes](./Images/distrib_panne.png)

![Distribution des pannes](./Images/distrib_erreur.png)

La première sera qu'une classe est largement majoritaire par rapport à l'autre cela implique qu'entrainer un modèle sur une base de données comme celle-ci aura tendance à biaiser le résultat. On aura un modèle qui ne sera pas vraiment performant dans les prédictions. Deuxièmement, dans les types d'erreurs, c'est très inégale aussi donc s'entrainer sur ce modèle pourrait aussi faire en sorte que le modèle s'entraine et finisse par repérer qu'un seul type d'erreur. Finalement, nous avions aussi un problème de multi-labeling car il pouvait y avoir plusieurs erreurs pour une seule et même machine, c'étaient des cas minoritaires donc nous avons préféré les enlever quand nous avions besoin d'éviter le cas du multi-labeling. 

En partant de cela, nous avons quand même essayer d'entrainer et de tester un modèle.

![Distribution des pannes](./Images/accuraccy1.png)

![Distribution des pannes](./Images/loss1.png)

D'après ces résultats, notre modèle semble tout bonnement parfait et ne présente aucune faille, mais pour en être sur nous avons étudié les matrice de confusion liées à chaque classe.

![Distribution des pannes](./Images/confusion1_0.png)
![Distribution des pannes](./Images/confusion1_1.png)

On remarque alors que notre modèle prédit que toutes les machines vont soit dans le oui, soit le non, mais sans nuance entre les deux, cela veut dire qu'il ne prédit pas vraiment, même pas du tout. Il a mal était entrainé et considère juste que soit toutes les machines marches sans l'erreur, soit qu'elles ont toutes l'erreurs.

Pour palier à ce problème de déséquilibre des classes de notre base d'entrainement, nous avons essayé d'utiliser deux méthodes afin de balancer tout cela.
La première était l'utilisation de la fonction smote qui permettait de créer artificiellement des cas pour les classes minoritaies. La deuxième était l'utilisation d'undersampling afin de réduire la taille des classes qui justement était bien plus majoritaire.
Ensuite nous avons retesté notre modèle et avons obtenu des résultats bien différents.
[insérer courbe accuraccy loss]
Notre modèle obtient maintenant une accuraccy bien moins parfaite que précemment avec notamment un peu d'overfitting. Pour vérifier la pertinence de notre équilibrage de classe, nous avons aussi visonner les matrice de confusion des classes comme tout à l'heure.

![Distribution des pannes](./Images/confusion2_1.png)
![Distribution des pannes](./Images/confusion2_0.png)

Le résultat n'est toujours pas satisfaisant à 100% mais notre modèle permet déjà de prédire un peu plus. En effet, il ne met plus bêtement toutes les machines dans le même panier, il essaie réellement de les répartir, même si cela ne correspond pas toujours avec la réalité, le modèle commence à essayer de prédire.
Finalement, nous avons garder ce modèle afin de tester la suite sur la carte même s'il n'est clairement pas otpimisé et mériterait qu'on s'y penche encore un peu dessus.