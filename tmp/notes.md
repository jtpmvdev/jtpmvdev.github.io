Oui. Le problème vient probablement de cette partie :

const tableResult = await grist.docApi.applyUserActions([
  ["AddTable", targetTableId, columns]
]);

Dans l’API Grist, AddTable ne renvoie pas toujours un objet contenant table_id. De plus, l’identifiant réellement créé peut différer du nom demandé. Le repli sur targetTableId peut donc produire un mauvais identifiant, puis AddRecord échoue.

Il y a aussi deux problèmes fréquents :

    Le nom de table peut déjà exister.
    Les colonnes peuvent avoir des identifiants invalides ou identiques après nettoyage.

Remplace la partie allant de // 2. Créer la table jusqu’à la fin de l’import par cette version :

// 2. Vérifier si la table existe déjà
const tablesBefore = await grist.docApi.listTables();

if (tablesBefore.includes(targetTableId)) {
  throw new Error(
    `La table "${targetTableId}" existe déjà. Choisissez un autre nom.`
  );
}

// 3. Créer la table
const tableResult = await grist.docApi.applyUserActions([
  ["AddTable", targetTableId, columns]
]);

console.log("Résultat AddTable :", tableResult);

// Selon la version de Grist, AddTable peut renvoyer directement
// l'identifiant de la table ou un tableau contenant cet identifiant.
let createdTableId = targetTableId;

if (typeof tableResult === "string") {
  createdTableId = tableResult;
} else if (Array.isArray(tableResult)) {
  const result = tableResult[0];

  if (typeof result === "string") {
    createdTableId = result;
  } else if (result && typeof result === "object") {
    createdTableId =
      result.table_id ||
      result.tableId ||
      result.id ||
      targetTableId;
  }
}

// 4. Relire la liste des tables pour confirmer la création
const tablesAfter = await grist.docApi.listTables();

if (!tablesAfter.includes(createdTableId)) {
  // Si Grist a conservé le nom demandé, on l'utilise.
  if (tablesAfter.includes(targetTableId)) {
    createdTableId = targetTableId;
  } else {
    throw new Error(
      `La table n'a pas été créée. Tables disponibles : ${tablesAfter.join(", ")}`
    );
  }
}

console.log("Table créée :", createdTableId);

// 5. Préparer les actions d'insertion
const addRecordActions = csvRows.map(row => {
  const recordFields = {};

  csvHeaders.forEach((header, idx) => {
    const value = row[header];

    recordFields[columns[idx].id] = …

Je recommande également de modifier la génération des identifiants de colonnes. Remplace :

const baseId = header.trim().replace(/[^a-zA-Z0-9_]/g, "_") || `colonne_${index + 1}`;

par :

const baseId =
  header
    .trim()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/[^a-zA-Z0-9_]/g, "_")
    .replace(/^[^a-zA-Z_]+/, "")
    .replace(/_+/g, "_")
    .replace(/^_+|_+$/g, "") ||
  `colonne_${index + 1}`;

Et ajoute cette validation après la création de targetTableId :

if (!/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(targetTableId)) {
  showStatus(
    "Le nom de table doit commencer par une lettre ou un underscore et ne contenir que des lettres, chiffres et underscores.",
    "error"
  );
  return;
}

Enfin, vérifie que ton widget personnalisé demande bien :

grist.ready({ requiredAccess: "full" });

et que l’utilisateur possède réellement un accès en écriture au document. Sans accès complet, la création de table et l’ajout de lignes échoueront, même si le code est correct.
