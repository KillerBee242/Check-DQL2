# Check-DQL

## 1. Afficher les employés masculins dont le salaire net (salaire + commission) est supérieur ou égal à 8000, triés par ancienneté décroissante.
-- La table résultante doit inclure : Numéro d'employé, Prénom et Nom, Âge et Ancienneté.
```
SELECT
    EMPLOYEE_int AS "Numéro d'employé",
    FIRST_NAME AS "Prénom",
    LAST_NAME AS "Nom",
    TIMESTAMPDIFF(YEAR, BIRTH_DATE, CURDATE()) AS "Âge",
    TIMESTAMPDIFF(YEAR, HIRE_DATE, CURDATE()) AS "Ancienneté"
FROM
    EMPLOYEES
WHERE
    TITLE = 'Mr.' AND (SALARY + IFNULL(COMMISSION, 0)) >= 8000
ORDER BY
    HIRE_DATE ASC;
```

## 2. Afficher les produits répondant aux critères suivants :
-- (C1) la quantité est emballée en bouteille(s),
-- (C2) le troisième caractère du nom du produit est 't' ou 'T',
-- (C3) fournis par les fournisseurs 1, 2 ou 3,
-- (C4) le prix unitaire est compris entre 70 et 200, et
-- (C5) les unités commandées sont spécifiées (non nulles).
-- La table résultante doit inclure : numéro de produit, nom de produit, numéro de fournisseur, unités commandées et prix unitaire.
```
SELECT
    PRODUCT_REF AS "Numéro de produit",
    PRODUCT_NAME AS "Nom de produit",
    SUPPLIER_int AS "Numéro de fournisseur",
    UNITS_ON_ORDER AS "Unités commandées",
    UNIT_PRICE AS "Prix unitaire"
FROM
    PRODUCTS
WHERE
    QUANTITY LIKE '%bottle(s)%'
    AND (SUBSTRING(PRODUCT_NAME, 3, 1) = 't' OR SUBSTRING(PRODUCT_NAME, 3, 1) = 'T')
    AND SUPPLIER_int IN (1, 2, 3)
    AND UNIT_PRICE BETWEEN 70 AND 200
    AND UNITS_ON_ORDER IS NOT NULL;
```

## 3. Afficher les clients qui résident dans la même région que le fournisseur 1,
-- c'est-à-dire qu'ils partagent le même pays, la même ville et les trois derniers chiffres du code postal.
-- La requête doit utiliser une seule sous-requête.
-- La table résultante doit inclure toutes les colonnes de la table CUSTOMERS.
```
SELECT
    c.*
FROM
    CUSTOMERS c
WHERE
    (c.COUNTRY, c.CITY, SUBSTRING(c.POSTAL_CODE, -3)) IN (
        SELECT
            s.COUNTRY,
            s.CITY,
            SUBSTRING(s.POSTAL_CODE, -3)
        FROM
            SUPPLIERS s
        WHERE
            s.SUPPLIER_int = 1
    );
```

## 4. Pour chaque numéro de commande entre 10998 et 11003, faire ce qui suit :
-- - Afficher le nouveau taux de remise, qui doit être de 0 % si le montant total de la commande avant remise (prix unitaire * quantité) est compris entre 0 et 2000,
--   5 % si entre 2001 et 10000, 10 % si entre 10001 et 40000, 15 % si entre 40001 et 80000, et 20 % autrement.
-- - Afficher le message "appliquer ancien taux de remise" si le numéro de commande est compris entre 10000 et 10999, et "appliquer nouveau taux de remise" autrement.
-- La table résultante doit afficher les colonnes : numéro de commande, nouveau taux de remise et note d'application du taux de remise.
```
SELECT
    o.ORDER_int AS "Numéro de commande",
    CASE
        WHEN (od.UNIT_PRICE * od.QUANTITY) BETWEEN 0 AND 2000 THEN '0%'
        WHEN (od.UNIT_PRICE * od.QUANTITY) BETWEEN 2001 AND 10000 THEN '5%'
        WHEN (od.UNIT_PRICE * od.QUANTITY) BETWEEN 10001 AND 40000 THEN '10%'
        WHEN (od.UNIT_PRICE * od.QUANTITY) BETWEEN 40001 AND 80000 THEN '15%'
        ELSE '20%'
    END AS "Nouveau taux de remise",
    CASE
        WHEN o.ORDER_int BETWEEN 10000 AND 10999 THEN 'appliquer ancien taux de remise'
        ELSE 'appliquer nouveau taux de remise'
    END AS "Note d'application du taux de remise"
FROM
    ORDERS o
JOIN
    ORDER_DETAILS od ON o.ORDER_int = od.ORDER_int
WHERE
    o.ORDER_int BETWEEN 10998 AND 11003;
```

## 5. Afficher les fournisseurs de produits de boisson.
-- La table résultante doit afficher les colonnes : numéro de fournisseur, entreprise, adresse et numéro de téléphone.
```
SELECT DISTINCT
    s.SUPPLIER_int AS "Numéro de fournisseur",
    s.COMPANY AS "Entreprise",
    s.ADDRESS AS "Adresse",
    s.PHONE AS "Numéro de téléphone"
FROM
    SUPPLIERS s
JOIN
    PRODUCTS p ON s.SUPPLIER_int = p.SUPPLIER_int
JOIN
    CATEGORIES c ON p.CATEGORY_CODE = c.CATEGORY_CODE
WHERE
    c.CATEGORY_NAME = 'Beverages';
```

## 6. Afficher les clients de Berlin qui ont commandé au plus 1 (0 ou 1) produit de dessert.
-- La table résultante doit afficher la colonne : code client.
```
SELECT
    c.CUSTOMER_CODE AS "Code client"
FROM
    CUSTOMERS c
WHERE
    c.CITY = 'Berlin'
    AND c.CUSTOMER_CODE IN (
        SELECT
            o.CUSTOMER_CODE
        FROM
            ORDERS o
        JOIN
            ORDER_DETAILS od ON o.ORDER_int = od.ORDER_int
        JOIN
            PRODUCTS p ON od.PRODUCT_REF = p.PRODUCT_REF
        JOIN
            CATEGORIES cat ON p.CATEGORY_CODE = cat.CATEGORY_CODE
        WHERE
            cat.CATEGORY_NAME = 'Desserts'
        GROUP BY
            o.CUSTOMER_CODE
        HAVING
            COUNT(DISTINCT p.PRODUCT_REF) <= 1
    )
UNION
SELECT
    c.CUSTOMER_CODE
FROM
    CUSTOMERS c
WHERE
    c.CITY = 'Berlin'
    AND c.CUSTOMER_CODE NOT IN (
        SELECT DISTINCT
            o.CUSTOMER_CODE
        FROM
            ORDERS o
        JOIN
            ORDER_DETAILS od ON o.ORDER_int = od.ORDER_int
        JOIN
            PRODUCTS p ON od.PRODUCT_REF = p.PRODUCT_REF
        JOIN
            CATEGORIES cat ON p.CATEGORY_CODE = cat.CATEGORY_CODE
        WHERE
            cat.CATEGORY_NAME = 'Desserts'
    );
```

## 7. Afficher les clients qui résident en France et le montant total des commandes qu'ils ont passées chaque lundi en avril 1998
-- (en considérant les clients qui n'ont pas encore passé de commande).
-- La table résultante doit afficher les colonnes : numéro de client, nom de l'entreprise, numéro de téléphone, montant total et pays.
```
SELECT
    C.CUSTOMER_CODE AS "Numéro de client",
    C.COMPANY AS "Nom de l'entreprise",
    C.PHONE AS "Numéro de téléphone",
    SUM(CASE
        WHEN DAYOFWEEK(O.ORDER_DATE) = 2 AND MONTH(O.ORDER_DATE) = 4 AND YEAR(O.ORDER_DATE) = 1998
        THEN OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)
        ELSE 0
    END) AS "Montant total",
    C.COUNTRY AS "Pays"
FROM
    CUSTOMERS C
LEFT JOIN
    ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
LEFT JOIN
    ORDER_DETAILS OD ON O.ORDER_int = OD.ORDER_int
WHERE
    C.COUNTRY = 'France'
GROUP BY
    C.CUSTOMER_CODE, C.COMPANY, C.PHONE, C.COUNTRY
ORDER BY
    C.CUSTOMER_CODE;
```

## 8. Afficher les clients qui ont commandé tous les produits.
-- La table résultante doit afficher les colonnes : code client, nom de l'entreprise et numéro de téléphone.
```
SELECT
    C.CUSTOMER_CODE AS "Code client",
    C.COMPANY AS "Nom de l'entreprise",
    C.PHONE AS "Numéro de téléphone"
FROM
    CUSTOMERS C
JOIN
    ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
JOIN
    ORDER_DETAILS OD ON O.ORDER_int = OD.ORDER_int
GROUP BY
    C.CUSTOMER_CODE, C.COMPANY, C.PHONE
HAVING
    COUNT(DISTINCT OD.PRODUCT_REF) = (SELECT COUNT(DISTINCT PRODUCT_REF) FROM PRODUCTS);
```

## 9. Afficher pour chaque client de France le nombre de commandes qu'il a passées.
-- La table résultante doit afficher les colonnes : code client et nombre de commandes.
```
SELECT
    C.CUSTOMER_CODE AS "Code client",
    COUNT(O.ORDER_int) AS "Nombre de commandes"
FROM
    CUSTOMERS C
LEFT JOIN
    ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
WHERE
    C.COUNTRY = 'France'
GROUP BY
    C.CUSTOMER_CODE
ORDER BY
    C.CUSTOMER_CODE;
```

## 10. Afficher le nombre de commandes passées en 1996, le nombre de commandes passées en 1997 et la différence entre ces deux nombres.
-- La table résultante doit afficher les colonnes : commandes en 1996, commandes en 1997 et Différence.
```
SELECT
    SUM(CASE WHEN YEAR(ORDER_DATE) = 1996 THEN 1 ELSE 0 END) AS "Commandes en 1996",
    SUM(CASE WHEN YEAR(ORDER_DATE) = 1997 THEN 1 ELSE 0 END) AS "Commandes en 1997",
    SUM(CASE WHEN YEAR(ORDER_DATE) = 1997 THEN 1 ELSE 0 END) - SUM(CASE WHEN YEAR(ORDER_DATE) = 1996 THEN 1 ELSE 0 END) AS "Différence"
FROM
    ORDERS;
```
