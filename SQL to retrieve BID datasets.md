This piece of SQL will retrieve all datasets where the EML(xml) file contains a project ID that begins with the string 'BID'.


>SELECT (xpath('./dataset/project/@id', t1.eml))[1]::text AS eml, t1.dk, t1.dataset_title, t1.publisher FROM

`The xpath object navigates the xml hierarchy, returns an array and must be cast as text`

 >(
 >
 >SELECT convert_from(m.content, 'LATIN1')::xml AS eml, d.key AS dk, d.title as dataset_title, o.title AS publisher
 
 `The metadata content field is assumed to be LATIN1 and converted to the DB encoding. Otherwise UTF8 breaking characters will trip up the query.`
 
 >FROM node
JOIN organization o ON node.key = o.endorsing_node_key
JOIN dataset d ON o.key = d.publishing_organization_key
JOIN metadata m ON d.key = m.dataset_key WHERE 
d.deleted IS NULL AND encode(m.content, 'escape') !~ '^\\357\\273\\277|^\\377\\376'

`Regex ignores xml with BOM and UTF16LE BOM`

>)t1

>WHERE trim((xpath('./dataset/project/@id', t1.eml))[1]::text) LIKE '%BID%'
