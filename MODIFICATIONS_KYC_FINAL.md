# Modifications KYC - Version Finale avec Bouton Retry

## Date : 29 janvier 2026

## Objectif
Améliorer la section "Document Submission" avec des champs utilisateur, un effet de flou partiel après soumission, et un bouton Retry pour recommencer sans recharger la page.

## Modifications apportées

### 1. Agrandissement de la section
- Largeur maximale : **600px → 750px**
- Padding : **40px → 50px**

### 2. Ajout des champs utilisateur

**Name** (Nom complet)
- Type : text
- Placeholder : "Enter your full name"
- Attribut : required

**N°** (Numéro d'identification)
- Type : text
- Placeholder : "Enter your identification number"
- Attribut : required

### 3. Structure HTML avec zones distinctes

```html
<form id="kyc-form">
    <!-- Partie qui sera FLOUE après soumission -->
    <div id="kyc-form-content">
        - Champs Name et N°
        - Zone de paste pour crypto/IBAN
        - Bouton Paste
        - Message d'aide
    </div>

    <!-- Partie qui reste NETTE après soumission -->
    <div class="kyc-actions">
        - Bouton Submit for Verification
        - Bouton Retry (caché par défaut)
    </div>
    <div id="kyc-status">
        - Message de confirmation
    </div>
</form>
```

### 4. Bouton Retry (NOUVEAU !)

**Caractéristiques :**
- Couleur : Orange (#f59e0b)
- Icône : 🔄
- Texte : "Retry"
- Caché par défaut
- Apparaît uniquement après la soumission
- Permet de réinitialiser le formulaire sans recharger la page

**CSS :**
```css
.kyc-retry {
    border: none;
    border-radius: 10px;
    padding: 15px 35px;
    font-size: 1.1rem;
    font-weight: 600;
    cursor: pointer;
    background: #f59e0b;
    color: #fff;
    display: none;
    align-items: center;
    gap: 10px;
    box-shadow: 0 6px 20px rgba(245, 158, 11, 0.3);
    transition: all 0.2s ease;
    margin-left: 10px;
}

.kyc-retry:hover {
    background: #d97706;
    transform: translateY(-2px);
}

.kyc-retry.visible {
    display: inline-flex;
}
```

**Fonction de réinitialisation :**
```javascript
function resetForm() {
    // Retirer le flou
    const kycFormContent = document.getElementById("kyc-form-content");
    kycFormContent.classList.remove("blurred");

    // Réinitialiser les champs
    document.getElementById("user-name").value = "";
    document.getElementById("user-number").value = "";
    docInput.value = "";
    pasteField.textContent = "Click \"Paste\" to submit your crypto address or iban";
    pasteField.classList.remove("validated");

    // Réinitialiser le bouton submit
    submitBtn.disabled = true;
    submitBtn.textContent = "📤 Submit for Verification";
    submitBtn.style.background = "#8f2829";
    submitBtn.style.color = "#fff";
    submitBtn.style.boxShadow = "0 6px 20px rgba(143, 40, 41, 0.3)";

    // Cacher le bouton Retry et le message de statut
    retryBtn.classList.remove("visible");
    statusEl.style.display = "none";
    statusEl.textContent = "";

    // Réinitialiser la variable de validation
    hasValidData = false;
}
```

### 5. Effet de flou partiel

**DEVIENT FLOU** après soumission :
- ✅ Champs Name et N°
- ✅ Zone de paste crypto/IBAN
- ✅ Bouton Paste
- ✅ Message d'aide

**RESTE NET** après soumission :
- ✅ Bouton "✓ Submitted" (vert)
- ✅ Bouton "🔄 Retry" (orange) - NOUVEAU !
- ✅ Message de confirmation (vert clair)
- ✅ Lien "← Back to Dashboard Menu"

## Flux utilisateur complet

### Étape 1 : Remplissage
```
┌─────────────────────────────────────────┐
│      Document Submission                │
├─────────────────────────────────────────┤
│  Name                                   │
│  [John Doe...........................]  │
│                                         │
│  N°                                     │
│  [123456............................]  │
│                                         │
│  [0x1234...abcd.......] [Paste]        │
│                                         │
│  [📤 Submit for Verification]          │
└─────────────────────────────────────────┘
```

### Étape 2 : Après soumission
```
┌─────────────────────────────────────────┐
│      Document Submission                │
├─────────────────────────────────────────┤
│  [████ FLOU ████████████████████]       │ ← Champs
│  [████ FLOU ████████████████████]       │ ← Zone paste
│  [████ FLOU ████]                       │ ← Bouton Paste
│                                         │
│  [✓ Submitted] [🔄 Retry]              │ ← NET
│                                         │
│  ✅ Votre soumission a été reçue...     │ ← NET
│                                         │
│  ← Back to Dashboard Menu               │ ← NET
└─────────────────────────────────────────┘
```

### Étape 3 : Après clic sur Retry
```
┌─────────────────────────────────────────┐
│      Document Submission                │
├─────────────────────────────────────────┤
│  Name                                   │
│  [....................................]  │ ← Vide et net
│                                         │
│  N°                                     │
│  [....................................]  │ ← Vide et net
│                                         │
│  [Click "Paste"........] [Paste]       │ ← Réinitialisé
│                                         │
│  [📤 Submit for Verification]          │ ← Désactivé
└─────────────────────────────────────────┘
```

## Avantages

1. **Pas de rechargement** : L'utilisateur peut recommencer sans recharger la page
2. **UX fluide** : Transition douce entre les états
3. **Feedback visuel clair** : 
   - Vert = Succès
   - Orange = Recommencer
   - Flou = Verrouillé
4. **Économie de temps** : Pas besoin de naviguer ou recharger
5. **Intuitivité** : Le bouton Retry apparaît au bon moment

## Fichier modifié

- `account/kyc/index.html`
  - HTML : Ajout du bouton Retry
  - CSS : Styles pour `.kyc-retry`
  - JavaScript : Fonction `resetForm()` et événement du bouton
