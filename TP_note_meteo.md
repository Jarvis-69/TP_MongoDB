Jeu de données: Téléchargez ou générez un jeu de données de stations météorologiques, qui incluent la date, la température, la pression atmosphérique, etc.

Préparation des données:
a. Importez les données de stations météorologiques dans MongoDB en utilisant la commande mongoimport. -->
b. Assurez-vous que les données sont bien structurées et propres pour une utilisation ultérieure.

Indexation avec MongoDB:
a. Créez un index sur le champ de la date pour améliorer les performances de la recherche. Utilisez la méthode createIndex ().
    
    db.Données.createIndex({DATE : 1})
    'DATE_1'
   

b. Vérifiez que l'index a été créé en utilisant la méthode listIndexes ().
    
    db.Données.getIndexes()
    [
    { v: 2, key: { _id: 1 }, name: '_id_' },
    { v: 2, key: { DATE: 1 }, name: 'DATE_1' }
    ]
   
Requêtes MongoDB:
a. Recherchez les stations météorologiques qui ont enregistré une température supérieure à 25° <!-- pendant les mois d'été (juin à août).  --> Utilisez la méthode find () et les opérateurs de comparaison pour trouver les documents qui correspondent à vos critères.
    
    db.Données.find({"TX" : {$gte : 25}}, {"_id" : 1})
    {
    _id: ObjectId("63da485d9c442146760ff4ce")
    }
    {
    _id: ObjectId("63da485d9c442146760ff4cf")
    }
    {
    _id: ObjectId("63da485d9c442146760ff4d0")
    }

Je n'arrive pas à changer la date comme il le faut pour l'exercice, mais la requète donnerai ça :
            
    db.Données.find({"TX" : {$elemMatch : {$and : [{"DATE" : {$gte : "2020/06/01"} }, {"DATE" : {$lte :"2020/08/31"} }, {$gte : "25" }}, {"_id" : 1})

    db.Données.find({"readings": {$elemMatch: {"TX": {$gt: 25}, "DATE": {$gte: new Date("2020/06/01"), $lte: new Date("2020/08/31")}}}}, {"_id" : 1})

b. Triez les stations météorologiques par HAUTEUR DE PRÉCIPITATIONS QUOTIDIENNE, du plus élevé au plus bas. Utilisez la méthode sort () pour trier les résultats.
    
    db.Données.find({}, {"RR" : 1}).sort({"RR" : -1})
    {
    _id: ObjectId("63da48889c4421467610886f"),
    RR: '90,5'
    }
    {
    _id: ObjectId("63da485d9c442146760ff5dc"),
    RR: '9,9'
    }
    {
    _id: ObjectId("63da485d9c442146760ff83c"),
    RR: '9,9'
    }
    {
    _id: ObjectId("63da48639c44214676100ad5"),
    RR: '9,9'
    }
    {
    _id: ObjectId("63da48639c44214676100c33"),
    RR: '9,9'
    }
   

Framework d'agrégation:
a. Calculez la température moyenne par station météorologique pour chaque mois de l'année. Utilisez le framework d'agrégation de MongoDB pour effectuer des calculs sur les données et grouper les données par mois.

    db.Données.aggregate([{$unwind: "$TM"}, {$group: {_id: {station: "$NOM", month: {$month: "$DATE"}}, avgTemperature: {$avg: "$TM"}}}])
    
    {
    _id: {
        station: 'GAP',
        month: 7
    },
    avgTemperature: 8.5
    }
    {
    _id: {
        station: 'AGEN-LA GARENNE',
        month: 9
    },
    avgTemperature: 12.8125
    }
    {
    _id: {
        station: 'POITIERS-BIARD',
        month: 8
    },
    avgTemperature: 11.4375
    }


b. Trouvez la station météorologique qui a enregistré la plus haute température en été.  Utilisez le framework d'agrégation de MongoDB pour effectuer des calculs sur les données et trouver la valeur maximale.

    db.Données.aggregate([{$unwind: "$TM"}, {$match: {"DATE": {"$gte": new Date("2020/06/01"),"$lte": new Date("2020/08/31")}}}, {$group: {_id: "$NOM",maxTemperature: { $max: "$TM" }}},{$sort: {maxTemperature: -1}},{$limit: 1}]) 
    
    {
    _id: 'ANGOULEME - BRIE - CHAMPNIERS',
    maxTemperature: 30
    }


Export de la base de données:
a. Exportez les résultats des requêtes dans un fichier CSV pour un usage ultérieur. Utilisez la commande mongoexport pour exporter des données de MongoDB.