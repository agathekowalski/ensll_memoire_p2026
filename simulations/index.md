---
layout: default
title: Simulation acoustique

---
# Simulation acoustique du théâtre d'Argentomagus 
 
## 1 - Calibrations des coefficients d'absorption et de diffusion des matériaux 
Une campagne de mesure menée dans le théâtre analogue de Syracuse, permet de viser un temps de réverbération moyen pour le théâtre d'Argentomagus. 

L'estimation du volume du modèle 3D du théâtre et la surface occupée par chacun de ses matériaux est possible dans _Blender_ au moyen d'un script _Python_ : 

### Calcul du volume total du modèle (appelé _mesh_) : 

```python
import bpy
import bmesh
from mathutils import Vector

def mesh_volume(obj):
    # Get evaluated mesh (includes modifiers)
    dg = bpy.context.evaluated_depsgraph_get()
    eval_obj = obj.evaluated_get(dg)
    mesh = eval_obj.to_mesh()

    # Ensure triangulation for accurate calculation
    bm = bmesh.new()
    bm.from_mesh(mesh)
    bmesh.ops.triangulate(bm, faces=bm.faces)
    bm.to_mesh(mesh)
    bm.free()

    volume = 0.0
    verts = mesh.vertices

    for poly in mesh.polygons:
        # Poly is now always a triangle thanks to triangulation
        v1, v2, v3 = (verts[i].co for i in poly.vertices)
        volume += v1.cross(v2).dot(v3)

    eval_obj.to_mesh_clear()

    return abs(volume) / 6.0

obj = bpy.context.active_object
print("Volume:", mesh_volume(obj))
```

On obtient alors une valeur en m³ : 

```python
Volume: 20589.84
```

### Calcul des surfaces occupées par chaque matériau (appelés _material_) :

```python
import bpy
import bmesh
from mathutils import Vector

obj = bpy.context.active_object
mesh = obj.data

bm = bmesh.new()
bm.from_mesh(mesh)
bm.faces.ensure_lookup_table()

material_areas = {}

for face in bm.faces:
    mat_index = face.material_index
    area = face.calc_area()
    material_areas[mat_index] = material_areas.get(mat_index, 0) + area

bm.free()

print("Surface area by material index:")
for mat_index, area in material_areas.items():
    mat_name = mesh.materials[mat_index].name if mesh.materials[mat_index] else "None"
    print(f"{mat_name}: {area:.4f} m2")
```

On obtient alors des valeurs en m² : 

```python
Surface area by material index:
100% absorbent: 3 465,0182 m2
Pierre Poreuse (Argentomagus): 1391.9365 m2
Mur de scene (Argentomagus): 27.3457 m2
Marbre frons pulpiti (Argentomagus): 29.4726 m2
Pilier (Argentomagus): 32.2453 m2
CR4_parquet (BRAS): 275.8170 m2
Audience : 2932.8210 m2
```


### Estimation du temps de réverbération moyen par bande de fréquences

On calcule alors un temps de réverbération théorique du modèle 3D, en fonction des dimensions obtenues et des coefficients d'absorption appliqués aux matériaux. La formule de Eyring est incluse dans le logiciel _open-source_ _pyroomacoustics_, lui-même importé au début du script. 

```python
import pyroomacoustics as pra 
from pyroomacoustics.parameters import air_absorption_table
import pandas as pd
import numpy 
import os

scaling_factor = 1.4 
# différence de diamètre entre Syracuse et Argentomagus 

m = air_absorption_table["20C_30-50%"]

script_dir = os.path.dirname(os.path.abspath(__file__))
csv_path = os.path.join(script_dir, "surfaces_materiaux_calibration.csv")

df = pd.read_csv(csv_path, sep=";")
#print(df)

frequencies = ['125Hz', '250Hz', '500Hz', '1000Hz', '2000Hz', '4000Hz', '8000Hz']
surfaces = df['Surface occupee (m2)']

# a*S par bande 
a_S = df[frequencies].multiply(surfaces, axis='index')

# somme des a*S
sum_a_S = a_S.sum()

S_total = surfaces.sum()

V_total = 20589.84 


# a par bande
a_coefficient = sum_a_S.values / S_total

for i, f in enumerate(frequencies) :
    eyring_syracuse = pra.rt60_eyring(S_total*(scaling_factor)**2,V_total*(scaling_factor)**3,a_coefficient[i],m[i],343)
    print(f, eyring_syracuse)
```
Grâce au facteur multiplicateur de **1.4** appliqué compensant la différence de taille entre les théâtres d'Argentomagus et de Syracuse, on peut prévoir une surestimation ou non du temps de réverbération par rapport aux mesures de Syracuse. On peut alors ajuster en fonction les coefficients de d'absorption et de diffusion des matériaux. 

On obtient des premiers temps de réverbération théoriques en secondes, par bande de fréquences : 

```python
125Hz 0.9612675076840118
250Hz 0.9587378998535472
500Hz 0.9652616418154282
1000Hz 0.9666220852043232
2000Hz 0.9852116816753088
4000Hz 1.076935900755007
8000Hz 1.7690107569556643
```

On trouve un temps de réverbération moyen de **0,96s** entre 500 Hz et 1000 Hz. Le temps cible issu des mesures du théâtre de Syracuse étant de **0,81s**, on augmente les coefficients d'absorption des matériaux majoritaires (ici la pierre poreuse et les gradins). De plus, les coefficients de diffusion sont relevés pour amoindrir la contribution de rayons « stationnaires » qui surestiment l'approximation du temps de réverbération.

On obtient donc une seconde banque de temps de réverbération par bande de fréquence : 

```python
125Hz 0.8545529356900029
250Hz 0.8478722506507228
500Hz 0.8444287953227818
1000Hz 0.8406911528811933
2000Hz 0.8900740918502903
4000Hz 0.9680336413059126
8000Hz 1.4935438193729154
```

Le temps de réverbération moyen de **0,84 s** calculé entre 500 Hz et 1000 Hz paraît plus cohérent en comparaison des mesures réalisées au théâtre de Syracuse. 


## 2 - Visualisation et écoute d'une réponse impulsionnelle obtenue

Voici une visualisation interactive des paramètres acoustiques issus d'une réponse impulsionnelle simulée dans le modèle 3D.
On choisit la simulation dont le temps de réverbération est le plus proche de la moyenne obtenue (ici de **0.86s**.
Les différents indices sont décrits en Annexes A (Indices de mesure de l'acoustique d'une salle) du mémoire. 

La simulation est effectuée avec **2 millions de rayons** et une surface réceptrice d'un **diamètre de 0,50 mètres**. 

<div style="width: 1200px; height: 800px; margin: 1rem 0; border: 1px solid #eee; border-radius: 5px;">
    <iframe
        src="{{ site.baseurl }}/assets/interactive/RIR_config1_listener3_source1.html"
        width="100%"
        height="100%"
        frameborder="0"
        style="border: none;">
    </iframe>
</div>

On montre sur l' image ci-dessous les positions de source et de récepteur correspondantes dans le modèle : 

<img
    src="{{ site.baseurl }}/assets/images/position_S_R.png"
    alt="Positions de la _source_ et du _listener_ dans le modèle 3D."
    style="width: 100%; height: 100%; max-width: 800px;border-radius: 5px; box-shadow: 0 2px 5px rgba(0,0,0,0.1);">


Voici à présent un **extrait audio** de cette réponse impulsionnelle convoluée avec un son de voix puis avec un extrait de la balade sonore : 

<audio controls>
  <source src="{{ site.baseurl }}/assets/sons/exemple_conv_IR.mp3' | relative_url  }}" type="audio/mpeg">
</audio>

## 3 - Cartographie des réflexions dans le modèle

Voici enfin une vidéo **cartographiant les réflexions en ambisonique**, pour les positions de source et de récepteur précédentes.
Elle est générée grâce aux outils internes de l'entreprise *Noise Makers*. Plus la tâche est rouge, plus l'amplitude de la réfléxion est élevée.

Un visuel équirectangulaire du modèle 3D est rendu à la position exacte de la source qui a permis d'obtenir la réponse impulsionnelle.
En superposant sur cette image les réflexions, on peut visualiser la provenance des réflexions entendues, ici principalement sur
la *cavea* inférieure.

<video
    controls
    width="100%"
    style="border-radius: 5px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin: 1rem 0;">
    <source src="{{ site.baseurl }}/assets/videos/visu1.mp4" type="video/mp4">
</video>

<br>

Voici l'emplacement d'un second couple source/récepteur dans l'espace modélisé :

<img
    src="{{ site.baseurl }}/assets/images/position_S_R_2.png"
    alt="Positions de la _source_ et du _listener_ dans le modèle 3D."
    style="width: 100%; max-width: 800px; height: auto; border-radius: 5px; box-shadow: 0 2px 5px rgba(0,0,0,0.1);">

De la même manière, on peut visualiser la position des premières réflexions.
On retrouve une réflexion précoce sur le mur de scène et l'orchestre.

<video
    controls
    width="100%"
    style="border-radius: 5px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin: 1rem 0;">
    <source src="{{ site.baseurl }}/assets/videos/visu2.mp4" type="video/mp4">
</video>

Les réflexions provenant des côtés de la *cavea* sont entendues peu après, ce qui semble cohérent par rapport à la vidéo.

Voici enfin les graphiques de la réponse impulsionnelle et de la courbe de décroissance correspondantes : 

<div style="width: 1200px; height: 800px; margin: 1rem 0; border: 1px solid #eee; border-radius: 5px;">
    <iframe
        src="{{ site.baseurl }}/assets/interactive/RIR_config1_listener1_source1.html"
        width="100%"
        height="100%"
        frameborder="0"
        style="border: none;">
    </iframe>
</div>
