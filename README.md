# Guide Sonarr — Profil `MULTI VF Light`

Configuration Sonarr & Radarr basée sur la logique Custom Formats, s'adapte bien aux films et aux séries.
Vous pouvez interchanger le VFQ et VFF selon votre région ou langue préféré.

## Objectif

Le profil vise à :

- privilégier les releases françaises ;
- exiger une présence française et de la langue originale ;
- favoriser VFQ et VF2 ;
- accepter VFI ;
- laisser VFF neutre ;
- donner un léger avantage au 2160p ;
- éviter les épisodes trop petits selon la résolution ;
- exclure les releases non françaises ;
- exclure AV1 pour le moment.

---

## 1. Custom Formats

Créer les Custom Formats suivants dans :

**Settings → Custom Formats**

### 1.1 `Language: Not French`

**Condition :** `Language`

- Language: `French`
- `Except Language`: activé
- `Nier`: désactivé
- `Obligatoire`: désactivé

**Score : `-10000`**

Cette condition pénalise les releases que Sonarr identifie comme n'étant pas en français.

---

### 1.2 `Language: Original + French`

**Deux conditions `Language`, toutes deux obligatoires :**

1. `French` → **Obligatoire**
2. `Original` → **Obligatoire**

**Score : `+500`**

Cela constitue la base du score recherché.

---

### 1.3 `VF2`

**Condition :** `Release Title`

Regex :

```regex
(?i)\bVF2\b
```

**Score : `+101`**

---

### 1.4 `VFQ`

**Condition :** `Release Title`

Regex :

```regex
(?i)\bVFQ\b
```

**Score : `+101`**

---

### 1.5 `VFI`

**Condition :** `Release Title`

Regex :

```regex
(?i)\bVFI\b
```

**Score : `+50`**

---

### 1.6 `VFF`

**Condition :** `Release Title`

Regex :

```regex
(?i)\bVFF\b
```

**Score : `0`**

---

### 1.7 `2160p`

**Condition :** `Résolution`

- Résolution : `2160p`
- Nier : désactivé
- Obligatoire : désactivé

**Score : `+1`**

Le 2160p obtient seulement un petit bonus afin de le favoriser sans écraser les préférences linguistiques.

---

### 1.8 `1080p Max 1.5GB`

Deux conditions, toutes deux obligatoires :

#### Résolution
- `1080p`
- Obligatoire

#### Size
- Minimum : `0 GB`
- Maximum : `1.5 GB`
- Obligatoire

**Score : `-10000`**

---

### 1.9 `2160p Max 2.5GB`

Deux conditions, toutes deux obligatoires :

#### Résolution
- `2160p`
- Obligatoire

#### Size
- Minimum : `0 GB`
- Maximum : `2.5 GB`
- Obligatoire

**Score : `-10000`**

---

### 1.10 `Codec: AV1`

Sonarr ne proposant pas de condition Codec native dans l'interface utilisée, utiliser `Release Title`.

**Condition :** `Release Title`

Regex :

```regex
(?i)\bAV1\b
```

**Score : `-10000`**

> Cette règle est volontairement utilisée pour exclure AV1 pour le moment. Une réévaluation pourra être faite ultérieurement selon les modèles de Fire TV utilisés par les clients Plex.

---

## 2. Profil de qualité

Créer un nouveau profil dans :

**Settings → Profiles → Quality Profiles → Add New Profile**

Nom utilisé :

```text
MULTI VF Light
```

### Paramètres

- **Mises à niveau autorisées :** activé
- **Qualités autorisées :** groupe `1080p & 2160p`
- **Mise à niveau jusqu'à :** `1080p & 2160p`
- **Score minimum de format personnalisé :** `500`
- **Mise à niveau jusqu'au score de format personnalisé :** `602`
- **Minimum Custom Format Score Increment :** `1`

### Scores du profil

| Custom Format | Score |
|---|---:|
| Language: Not French | -10000 |
| Language: Original + French | +500 |
| VF2 | +101 |
| VFQ | +101 |
| VFI | +50 |
| VFF | 0 |
| 2160p | +1 |
| 1080p Max 1.5GB | -10000 |
| 2160p Max 2.5GB | -10000 |
| Codec: AV1 | -10000 |

---

## 3. Comprendre les scores

La cible maximale actuelle est **602** :

```text
Original + French     500
VFQ ou VF2            101
2160p                   1
-------------------------
Total                  602
```

Exemples :

| Release | Score théorique |
|---|---:|
| VFF | 0 |
| VFI | 50 |
| VFQ | 101 |
| VF2 | 101 |
| Original + French | 500 |
| Original + French + VFI | 550 |
| Original + French + VFQ | 601 |
| Original + French + VF2 | 601 |
| Original + French + VFQ + 2160p | 602 |
| Original + French + VF2 + 2160p | 602 |

Une release doit atteindre au minimum **500** pour être acceptable par le profil.

Le score **602** représente actuellement la cible maximale souhaitée.

---

## 4. Pourquoi les grosses pénalités `-10000` ?

Les Custom Formats suivants utilisent `-10000` :

- `Language: Not French`
- `1080p Max 1.5GB`
- `2160p Max 2.5GB`
- `Codec: AV1`

Avec un seuil minimum de **500**, une release qui reçoit une de ces pénalités ne devrait pas satisfaire le profil, même si elle possède d'autres bonus.

---

## 5. Appliquer le profil à toutes les séries

Pour appliquer `MULTI VF Light` aux séries existantes :

1. Aller dans **Series**.
2. Ouvrir **Mass Editor / Éditeur en masse**.
3. Sélectionner toutes les séries voulues.
4. Choisir **Quality Profile**.
5. Sélectionner `MULTI VF Light`.
6. Enregistrer.

Le changement de profil ne signifie pas nécessairement que tous les épisodes existants seront immédiatement recherchés à nouveau. Pour identifier les épisodes qui n'atteignent pas le cutoff, utiliser les fonctions de recherche/cutoff de Sonarr.

---

## 6. Recherche interactive

Après configuration, il est recommandé de tester quelques séries avec :

**Series → série → Interactive Search**

Vérifier notamment :

- la langue détectée ;
- les mentions VFQ/VF2/VFI/VFF ;
- la résolution ;
- la taille ;
- le score Custom Format ;
- la présence éventuelle d'AV1.

Cela permet de confirmer que les releases réelles sont interprétées comme prévu.

---

## 7. Philosophie de la configuration

La priorité est :

1. **Français + langue originale**
2. **VFQ / VF2**
3. **VFI**
4. **VFF**
5. **2160p comme léger avantage**
6. **Respect des limites de taille**
7. **Pas d'AV1 pour le moment**

Le but est de favoriser une release française de bonne qualité tout en évitant les releases non françaises, les fichiers trop petits pour leur résolution et l'AV1.

---

## 8. Évolution possible

Cette configuration peut être ajustée ultérieurement.

En particulier, le choix d'exclure AV1 pourra être réévalué après identification des modèles de Fire TV utilisés par les clients Plex. Un serveur équipé d'une Intel Arc A380 possède des capacités matérielles utiles pour le traitement vidéo moderne, mais le Direct Play côté client reste préférable lorsqu'il est possible.
