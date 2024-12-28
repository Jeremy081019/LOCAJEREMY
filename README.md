# LOCAJEREMY
# Dépôt GitHub : SPATIAL RENNES

## Structure du Projet

### Frontend
Le frontend sera basé sur HTML, CSS et JavaScript, en utilisant [Chart.js](https://www.chartjs.org/) pour les visualisations.

**Structure des fichiers du frontend :**
```
/frontend
  |-- index.html    # Interface principale pour le test
  |-- style.css     # Fichiers CSS pour la mise en page
  |-- script.js     # Logique principale côté client
```

### Backend
Le backend sera construit avec Python et Flask pour gérer les données de test et stocker les résultats. Une base de données SQLite sera utilisée pour persister les résultats.

**Structure des fichiers du backend :**
```
/backend
  |-- app.py        # Application Flask principale
  |-- models.py     # Gestion de la base de données
  |-- requirements.txt # Dépendances Python
  |-- templates/    # Gérer les pages HTML rendues par Flask
  |-- static/       # Ressources statiques (CSS, JS, Images)
```

### API
Une API REST sera fournie pour :
1. Enregistrer les réponses des patients.
2. Récupérer les résultats pour analyse.
3. Télécharger les résultats au format CSV.

**Exemples de routes API :**
- `POST /api/save-result` : Enregistrer une réponse.
- `GET /api/results` : Récupérer les résultats.
- `GET /api/download` : Télécharger un fichier CSV des résultats.

### Dépendances
- Flask
- SQLite
- Chart.js (Frontend)
- Bootstrap (Frontend)

---

## Exemple de Code

### Frontend - `script.js`
```javascript
// Sauvegarde les résultats dans le backend
function saveResult(selectedIndex, currentSpeakerIndex, adjustedError) {
    fetch('/api/save-result', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            speaker: currentSpeakerIndex + 1,
            selected: selectedIndex + 1,
            error: adjustedError,
        })
    }).then(response => {
        if (response.ok) {
            console.log('Résultat sauvegardé.');
        } else {
            console.error('Erreur lors de la sauvegarde.');
        }
    });
}
```

### Backend - `app.py`
```python
from flask import Flask, request, jsonify, send_file
from models import db, Result
import csv
from io import StringIO

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///results.db'
db.init_app(app)

@app.route('/api/save-result', methods=['POST'])
def save_result():
    data = request.json
    result = Result(speaker=data['speaker'], selected=data['selected'], error=data['error'])
    db.session.add(result)
    db.session.commit()
    return jsonify({'message': 'Résultat sauvegardé !'})

@app.route('/api/results', methods=['GET'])
def get_results():
    results = Result.query.all()
    return jsonify([r.to_dict() for r in results])

@app.route('/api/download', methods=['GET'])
def download_csv():
    results = Result.query.all()
    si = StringIO()
    writer = csv.writer(si)
    writer.writerow(['Speaker', 'Selected', 'Error'])
    for r in results:
        writer.writerow([r.speaker, r.selected, r.error])
    output = si.getvalue()
    si.close()
    return send_file(StringIO(output), mimetype='text/csv', as_attachment=True, download_name='results.csv')

if __name__ == '__main__':
    app.run(debug=True)
```

### Backend - `models.py`
```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Result(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    speaker = db.Column(db.Integer, nullable=False)
    selected = db.Column(db.Integer, nullable=False)
    error = db.Column(db.Integer, nullable=False)

    def to_dict(self):
        return {
            'id': self.id,
            'speaker': self.speaker,
            'selected': self.selected,
            'error': self.error,
        }
```

### Instructions pour Exécuter

1. **Installer les dépendances :**
    ```bash
    pip install -r backend/requirements.txt
    ```

2. **Initialiser la base de données :**
    ```bash
    flask shell
    >>> from models import db
    >>> db.create_all()
    ```

3. **Lancer le serveur :**
    ```bash
    flask run
    ```

4. **Ouvrir le frontend :**
    Ouvrez `index.html` dans un navigateur.
