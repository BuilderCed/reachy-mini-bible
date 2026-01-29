# Reachy Mini Bible

> **Source unique de vérité** pour le développement Reachy Mini avec Claude, Cursor et Python.

[![Docs Verify](https://github.com/BuilderCed/reachy-mini-bible/actions/workflows/docs-verify.yml/badge.svg)](https://github.com/BuilderCed/reachy-mini-bible/actions/workflows/docs-verify.yml)
[![SDK Version](https://img.shields.io/badge/SDK-1.2.13-blue)](https://pypi.org/project/reachy-mini/)
[![Python](https://img.shields.io/badge/Python-3.10--3.12-green)](https://python.org)

---

## Navigation rapide

| Document | Description | Temps de lecture |
|----------|-------------|------------------|
| [QUICK_REFERENCE](docs/QUICK_REFERENCE.md) | Cheat sheet SDK, commandes essentielles | 10 min |
| [MASTER_CHECKLIST](docs/MASTER_CHECKLIST.md) | Checklist sécurité + décision flowchart | 5 min |
| [SUPER_PROMPTS](docs/SUPER_PROMPTS.md) | Prompts prêts à coller dans Claude/ChatGPT | 15 min |
| [COMPLETE_GUIDE](docs/COMPLETE_GUIDE.md) | Référence exhaustive (1000+ lignes) | 60 min |
| [DOCUMENTATION_INDEX](docs/DOCUMENTATION_INDEX.md) | Index de navigation par sujet | 5 min |

---

## Quick Start

```bash
# Installation SDK
pip install reachy-mini

# Connexion (auto-détection USB/WiFi)
python3 << 'EOF'
from reachy_mini import ReachyMini
from reachy_mini.utils import create_head_pose

with ReachyMini() as mini:
    print(f"Connected! Battery: {mini.battery_level}%")
    mini.goto_target(
        head=create_head_pose(pitch=10, degrees=True),
        antennas=[0.5, -0.5],
        duration=1.0
    )
EOF
```

---

## Architecture cible

```
reachy-mini-bible/           # Ce repo (source unique)
├── docs/                    # Documentation complète
├── scripts/                 # Outils de vérification
└── .github/workflows/       # CI automatisée

Votre projet Reachy/
├── CLAUDE.md               # Contexte projet → pointe ici
├── .cursor/rules/          # Règles Cursor → réf. cette bible
└── src/                    # Votre code
```

---

## Spécifications SDK (v1.2.13)

### API de connexion

```python
from reachy_mini import ReachyMini

# Auto-détection (recommandé)
with ReachyMini() as mini:
    pass

# Mode explicite
with ReachyMini(connection_mode="localhost_only") as mini:  # USB/Lite
    pass

with ReachyMini(host="192.168.1.xxx") as mini:  # Wireless IP
    pass
```

### Limites de sécurité

| Joint | Min | Max |
|-------|-----|-----|
| Head Pitch/Roll | -40° | +40° |
| Head Yaw | -180° | +180° |
| Body Yaw | -160° | +160° |
| Delta Yaw (head-body) | - | 65° max |

### Versions hardware

| Aspect | Lite ($299) | Wireless ($449) |
|--------|-------------|------------------|
| Connexion | USB-C | Wi-Fi 6, Bluetooth 5.2 |
| Compute | Externe | Raspberry Pi 5 (8GB) |
| Microphones | 2 | 4 |
| IMU | Non | Oui |
| Batterie | Non | Oui |

---

## Clés et Tokens

### Séparation Claude vs Cursor

Pour tracer l'usage par outil, utilise **deux clés API Anthropic distinctes**:

| Usage | Clé | Stockage macOS |
|-------|-----|----------------|
| Claude Desktop / Apps Python | `ANTHROPIC_API_KEY` | Keychain: `claude_apps` |
| Cursor IDE | Clé dédiée | Cursor Settings > API |

**Commandes Keychain macOS:**

```bash
# Ajouter clé Claude Desktop / Apps
security add-generic-password \
  -a "claude_apps" \
  -s "ANTHROPIC_API_KEY" \
  -w "sk-ant-api03-XXXXX" \
  -T ""

# Récupérer la clé (dans scripts)
security find-generic-password -a "claude_apps" -s "ANTHROPIC_API_KEY" -w
```

### Token GitHub (pour automation)

Créer un **Fine-grained Personal Access Token** (compte personnel uniquement):

1. GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. **Token name:** `reachy-bible-automation`
3. **Expiration:** 90 jours (ou plus selon besoin)
4. **Resource owner:** Ton compte personnel (pas Mounki)
5. **Repository access:** Only select repositories → `reachy-mini-bible`
6. **Permissions:**
   - **Contents:** Read and write (push fichiers)
   - **Metadata:** Read (obligatoire)
   - **Actions:** Read (si monitoring workflows)

```bash
# Stocker le token GitHub
security add-generic-password \
  -a "github_reachy_bible" \
  -s "GITHUB_PAT" \
  -w "github_pat_XXXXX" \
  -T ""
```

---

## Routine de vérification

Le script `scripts/verify-docs.sh` vérifie:

1. Tous les fichiers référencés dans l'index existent
2. Liens internes valides
3. Version SDK documentée vs PyPI
4. Pas de références obsolètes

```bash
./scripts/verify-docs.sh
```

Le workflow GitHub Actions `.github/workflows/docs-verify.yml` exécute cette vérification sur chaque push/PR.

---

## Intégration Cursor

Dans ton projet Reachy, crée:

### `.cursor/rules/700-reachy-mini.mdc`

```markdown
---
description: Reachy Mini SDK standards
globs: "**/*.py"
alwaysApply: false
---

# Reachy Mini Development

Référence: https://github.com/BuilderCed/reachy-mini-bible

## SDK (v1.2.13)
- Connexion: `with ReachyMini() as mini:` (auto-détection)
- Mouvement: `mini.goto_target(head=..., antennas=..., duration=...)`
- Pose: `create_head_pose(pitch=10, degrees=True)`
- Média: `mini.media.get_frame()`, `mini.media.get_audio_sample()`

## Sécurité
- Limites auto-clampées par SDK
- Kids mode: speed 0.5x, duration min 1.5s, session max 30min
- Health check: battery > 20%, daemon accessible
```

### `CLAUDE.md` (racine projet)

```markdown
# Reachy Mini Project

Bible: https://github.com/BuilderCed/reachy-mini-bible

## Stack
- Python 3.10-3.12
- SDK: reachy-mini 1.2.13
- AI: Claude 3.5 Sonnet (LLM/VLM)

## Docs prioritaires
1. QUICK_REFERENCE.md - SDK cheat sheet
2. MASTER_CHECKLIST.md - Avant chaque session
3. SUPER_PROMPTS.md - Génération de code
```

---

## Ressources officielles

- **Documentation:** https://huggingface.co/docs/reachy_mini
- **GitHub:** https://github.com/pollen-robotics/reachy_mini
- **PyPI:** https://pypi.org/project/reachy-mini/
- **Discord:** https://discord.gg/2bAhWfXme9

---

## Changelog

Voir [CHANGELOG.md](docs/CHANGELOG.md)

---

**Dernière mise à jour:** Janvier 2026  
**SDK Version:** 1.2.13  
**Status:** Production Ready