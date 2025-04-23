# 📦 Exportación de Datos de Vtiger a CSV para Migración a Odoo

Este documento detalla la exportación de seis conjuntos clave de datos desde una base de datos Vtiger CRM, con el objetivo de migrar la información hacia Odoo. Cada sección incluye una descripción y la consulta SQL correspondiente.

---

## 1. 🏢 Compañías (Clientes)

Exporta la información principal de empresas clientes.

**Consulta SQL:**

```sql
SELECT acc.accountid, acc.accountname, acc.website, acc.phone, acc.email1, acc.email2, acc.otherphone, acc.fax,
       addr.bill_city, addr.bill_state, addr.bill_code, addr.bill_country
INTO OUTFILE '/var/lib/mysql-files/companies.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM vtiger_account acc
JOIN vtiger_accountbillads addr ON acc.accountid = addr.accountaddressid;
```

---

## 2. 👤 Contactos

Exporta personas de contacto asociadas a compañías.

**Consulta SQL:**

```sql
SELECT c.contactid, c.firstname, c.lastname, c.email, c.phone, c.mobile, c.accountid,
       a.accountname
INTO OUTFILE '/var/lib/mysql-files/contacts.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM vtiger_contactdetails c
LEFT JOIN vtiger_account a ON c.accountid = a.accountid;
```

---

## 3. 👨‍💼 Usuarios (Usuarios del sistema)

Lista de usuarios de Vtiger que interactúan con clientes.

**Consulta SQL:**

```sql
SELECT id, user_name, first_name, last_name, email1, phone_mobile
INTO OUTFILE '/var/lib/mysql-files/users.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM vtiger_users;
```

---

## 4. 💬 Interacciones (Notas y Comentarios)

Exporta comentarios hechos por los usuarios hacia clientes o contactos.

**Consulta SQL:**

```sql
SELECT 
    mc.commentcontent AS comment,
    mc.related_to,
    rel_ent.setype AS related_type,
    rel_ent.label AS related_name,
    u.user_name AS author,
    ent.createdtime
INTO OUTFILE '/var/lib/mysql-files/interactions.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM vtiger_modcomments mc
JOIN vtiger_crmentity ent ON mc.modcommentsid = ent.crmid
LEFT JOIN vtiger_users u ON mc.userid = u.id
LEFT JOIN vtiger_crmentity rel_ent ON mc.related_to = rel_ent.crmid
WHERE ent.deleted = 0;
```

---

## 5. 📎 Documentos Adjuntos

Exporta la lista de archivos adjuntos a registros.<br>
Actualmente se tiene  5GB de docuementos desde desde Agosto 2015, hasta Marzo 2025.

La estructura es: 2015/August/week2, 2015/August/week3, etc

**Consulta SQL:**

```sql
SELECT 
    att.attachmentsid, att.name, att.path, att.type, rel.crmid AS related_to,
    CONCAT('http://<tu_dominio>/storage/', att.attachmentsid, '_', att.name) AS download_link
INTO OUTFILE '/var/lib/mysql-files/documents.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM vtiger_attachments att
JOIN vtiger_seattachmentsrel rel ON att.attachmentsid = rel.attachmentsid;
```

---

## 6. 📈 Oportunidades (Potenciales de Venta)

Exporta oportunidades de negocio activas o históricas.

**Consulta SQL:**

```sql
SELECT 
    pot.potentialid,
    pot.potentialname AS opportunity_name,
    pot.amount,
    pot.closingdate,
    pot.sales_stage,
    pot.leadsource,
    pot.related_to,
    acc.accountname AS company_name,
    con.firstname AS contact_firstname,
    con.lastname AS contact_lastname,
    u.user_name AS assigned_to,
    ent.createdtime,
    ent.modifiedtime,
    pot.description
INTO OUTFILE '/var/lib/mysql-files/opportunities.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM vtiger_potential pot
JOIN vtiger_crmentity ent ON pot.potentialid = ent.crmid
LEFT JOIN vtiger_account acc ON pot.related_to = acc.accountid
LEFT JOIN vtiger_contactdetails con ON pot.related_to = con.contactid
LEFT JOIN vtiger_users u ON ent.smownerid = u.id
WHERE ent.deleted = 0;
```
