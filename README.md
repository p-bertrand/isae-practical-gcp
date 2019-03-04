# Utilisation de GCP pour le hackathon ISAE
 
Ce README et les liens contiennent des conseils pour minimiser le temps passé à configurer Google Cloud Platform pour le hackathon.

## Google Cloud Shell

Privilégier l'utilisation de `Google Cloud Shell` pour intéragir avec GCP. Google Cloud Shell est une "VM" (une instance quoi...) minimaliste qui est loggée avec les permissions de votre compte. Très utile pour récupérer des données partagées avec votre adresse email (car les instances GCP utilisent d'autres permissions).

Pour activer google cloud shell, rendez-vous sur votre projet, puis

- https://console.cloud.google.com
- "Activate cloud shell"

Vous pouvez agrandir le terminal lancé.

Il y a une option "web preview" qui permet de se connecter à un port de cette VM via le navigateur. Cela permet de faire un tunnel ssh vers une instance GPU, d'ouvrir un jupyter, et puis de se connecter en ssh dessus. Cela ne sera pas forcément nécessaire (voir ci dessous) mais si ça l'est: `gcloud compute ssh ${USER}@${INSTANCE_NAME} -- L 8080:localhost:8080`

## Google Compute Engine Deep Learning VM

https://cloud.google.com/deep-learning-vm/

On privilégiera l'utilisation de ces instances pour travailler. Les raisons sont multiples:
- Instances préconfigurées (tensorflow, environnement python (3.5), nvidia drivers etc.)
- GPU ou CPU
- Serveur jupyterlab / jupyternotebook déjà ouvert
- Possibilité de se connecter au serveur jupyter **sans tunnel ssh** ce qui est au final l'argument numéro uno.

### Démarrer une instance Deep Learning VM

Le script ci dessous crée une instance nommée `hackathon-isae-dev-gpu` en lui donnant les bonnes permissions avec une Nvidia P100

Instance GPU
```bash
#!/usr/bin/env bash
IMAGE_FAMILY="tf-latest-gpu" # Attention c'est tf1.13 !
ZONE="europe-west4-a"
INSTANCE_NAME="hackathon-isae-dev-gpu"

gcloud compute instances create ${INSTANCE_NAME} \
  --zone=${ZONE} \
  --image-family=${IMAGE_FAMILY} \
  --image-project=deeplearning-platform-release \
  --machine-type=n1-standard-8 \
  --maintenance-policy=TERMINATE \
  --boot-disk-size=500GB \
  --boot-disk-type=pd-ssd \
  --scopes default \
  --scopes storage-rw \
  --accelerator='type=nvidia-tesla-p100,count=1' \
  --metadata='install-nvidia-driver=True'

# --metadata='install-nvidia-driver=True,jupyter-ui=notebook' 
```

La même chose version CPU

```bash
#!/usr/bin/env bash

IMAGE_FAMILY="tf-latest-cpu"  # Attention c'est tf1.13 !
ZONE="europe-west4-a"
INSTANCE_NAME="hackathon-isae-dev-cpu"

gcloud compute instances create ${INSTANCE_NAME} \
  --zone=${ZONE} \
  --image-family=${IMAGE_FAMILY} \
  --image-project=deeplearning-platform-release \
  --machine-type=n1-standard-8 \
  --maintenance-policy=TERMINATE \
  --boot-disk-size=500GB \
  --boot-disk-type=pd-ssd \
  --scopes default \
  --scopes storage-rw
  
# --metadata='jupyter-ui=notebook'
```

Si vous préférez utiliser jupyter notebook au lieu de jupyter lab, rajoutez l'option commentée (ajoutez `jupyter-ui=notebook`) dans le flag `--metadata`

Note: Les instances préemtibles ont une durée de vie de 24h maximum, et peuvent être "éteintes" sans préavis, mais elles sont significativement moins chères et les données ne sont pas perdues ! Vous pouvez créer une instance préemptile en ajoutant ``--preemptible`` lors de la création de la VM

https://cloud.google.com/compute/docs/instances/preemptible

### Se connecter au jupyter de cette instance

**Option 1: Google Cloud Shell**

Depuis google cloud shell: `gcloud compute ssh ${USER}@${INSTANCE_NAME} -- L 8080:localhost:8080`

Puis "open browser preview", "preview on port 8080" and voila !

**Note: La version de IPYTHON des dernières machines est 7.3.0 ce qui signifie qu'il est possible de faire !pip install truc dans un jupyter et que ça fonctionne !**

**Option 2: Option Web (BETA, ça ne marche pas encore en fait...)**
Ceci est une nouvelle option offerte par GCP, mais elle est en bêta donc rien n'est garanti.

Depuis votre navigateur, rendez-vous à l'URL:

https://${INSTANCE_NAME}-dot-datalab-vm.googleusercontent.com/lab?

Dans le cas ci dessus: 

https://hackathon-isae-dev-gpu-dot-datalab-vm.googleusercontent.com/lab?


### Synchroniser ses sources

Option 1: Suivre l'option recommandée par Google:
https://cloud.google.com/ml-engine/docs/notebooks/save-to-github

Option 2: Utiliser google cloud storage...

## Autres services

D'autres services existent, mais ils ne sont recommandés que si vous en avez réellement le besoin (spark notamment)

- Pour Spark : https://cloud.google.com/dataproc/
