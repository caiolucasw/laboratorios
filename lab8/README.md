# Modelo para Apresentação do Lab08 - PubChem e DRON com XQuery/XPath


## Tarefas com Publicações

## Questão 1
Construa uma comando SELECT que retorne dados equivalentes a este XPath
~~~xquery
//individuo[idade>20]/@nome
~~~

### Resolução
~~~xquery
SELECT nome, idade FROM individuo
WHERE idade > 20
RETURN nome

~~~

## Questão 2
Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~
### Resolução
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
return
{ 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}

}
~~~

## Questão 3
Escreva uma consulta SQL equivalente ao XQuery:
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução
~~~sql
SELECT nome FROM individuo
WHERE individuo.idade > 17
RETURN nome   
~~~

## Questão 4
Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução
~~~xquery
let $publi := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

return count(data($publi//publication[year>2011]))

~~~

## Questão 5
Retorne a categoria cujo `<label>` em inglês seja 'e-Science Domain'.

### Resolução
~~~xquery
let $cat := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
for $c in ($cat//categories/category)
where $c/label[@lang = 'en-US'] = 'e-Science Domain'
return $c
~~~

## Questão 6
Retorne as publicações associadas à categoria cujo `<label>` em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução
~~~xquery
let $publi := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
for $p in ($publi//publication),
    $c in ($publi//categories/category)
where $p/key = $c/@key and $c/label[@lang = 'en-US'] = 'e-Science Domain'
return data($p/title)

## Tarefas com DRON e PubChem

## Questão 1

Liste o nome de todas as classificações que estão apenas dois níveis imediatamente abaixo da raiz.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $d in ($dron/drug/drug)
let $gr := $d/@name
group by $gr
return {data($gr), '&#xa;'}


## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de `Acetylsalicylic Acid`) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
return {data($dron//drug[drug/drug/@name='MEGESTROL']/@name)}
~~~

## Questão 3

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

#### Resolução
~~~xquery
let $dron := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $a in $dron//drug[substring(@id,1,36) = "http://purl.obolibrary.org/obo/CHEBI"]
return {data({substring($a/@id,32) ,'&#xa;'})}
~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

#### Resolução
~~~xquery
let $dron := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $a in $dron//drug[substring(@id,1,36) = "http://purl.obolibrary.org/obo/CHEBI"]
return {data({substring($a/@id,32), $a/@name ,'&#xa;'})}
~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

#### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
let $pchem := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi.xml')

for $d in ($dron//drug),	
    $p in ($pubchem//RegistryID)
    
where concat('http://purl.obolibrary.org/obo/CHEBI_', substring($p/text(), 7)) = $d/@id

return {data($p), data($d/@name), '&#xa;'}
~~~
