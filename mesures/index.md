---
layout: default
title: Mesures acoustiques

---
# Mesures acoustiques 

## Extraits audios des réponses impulsionnelles mesurées

Des mesures acoustiques ont été menées sur les vestiges du théâtre d'Argentomagus pour tenter d'approcher la réponse impulsionnelle du site en différents points. Un niveau de bruit de fond très élevé a toutefois imposé le rejet de certaines mesures pour lesquelles des artefacts temporels audibles apparaissaient après la déconvolution. 

Voici **deux extraits utilisés** pour juger la pertinence des réponses impulsionnelles obtenues (enregistrés en salle anéchoïque). 

Le premier est un court extrait de batterie : 
<audio controls>
  <source src="{{ 'mesures/shortDrums.mp3' | relative_url }}" type="audio/mpeg">
</audio>

Le second est un extrait vocal d'une voix masculine : 
<audio controls>
  <source src="{{ 'mesures/speech.mp3' }}" type="audio/mpeg">
</audio>

### Réponse impulsionnelle d'une mesure jugée pertinente

On donne a présent les mêmes sons, convolués par la réponse impulsionnelle issue de la **mesure 2, jugée correcte** : 

La batterie issue de la **convolution** : 
<audio controls>
  <source src="{{ '/mesures/shortDrums_m2.mp3' | relative_url }}" type="audio/mpeg">
</audio>

L'extrait vocal issu de la **convolution** : 
<audio controls>
  <source src="{{ '/mesures/speech_m2.mp3' | relative_url }}" type="audio/mpeg">
</audio>

On peut alors comparer ces extraits convolués par les mêmes sons, **enregistrés sur site** à la suite des trois balayages sinusoïdaux. En comparant les extraits convolués et mesurés, on peut juger les réponses impulsionnelles obtenues. 

L'extrait de batterie issu de la **mesure** sur site : 
<audio controls>
  <source src="{{ '/mesures/shortDrums_m2_mesure.mp3' | relative_url }}" type="audio/mpeg">
</audio>

L'extrait vocal issu de la **mesure** sur site : 
<audio controls>
  <source src="{{ '/mesures/speech_m2_mesure.mp3' | relative_url }}" type="audio/mpeg">
</audio>

On entend ici le **niveau très élevé de bruit ambiant**, qui a conduit à exclure certaines mesures, même après une opération de réduction de bruit. On entend tout de même que les deux signaux sont comparables. Une légère surestimation des bas médiums dans l'extrait issu de la convoltion pourra être corrigé au mixage par un outil de filtrage.

### Réponse impulsionnelle d'une mesure écartée 

Voici enfin les mêmes extraits convolués par la réponse impulsionnelle issue de la **mesure 8, jugée non pertinente** : 

La batterie : 
<audio controls>
  <source src="{{ '/mesures/shortDrums_m8.mp3' | relative_url }}" type="audio/mpeg">
</audio>

L'extrait vocal : 
<audio controls>
  <source src="{{ '/mesures/speech_m8.mp3' | relative_url }}" type="audio/mpeg">
</audio>

Des **artefacts numériques** sont audibles sur les transitoires, notamment dans l'extrait de batterie et les plausives de la voix masculines. Ils sont dus à un bruit ambiant trop élevé lors des mesures empêchant, pour certaines fréquences du balayage sinusoïdal, la discrimination entre le stimulus et le bruit extérieur. 



