![image](https://github.com/user-attachments/assets/afd8b227-d405-4159-95ee-152c77a958fe)


## DNS Question name
- Chercher dans champ : dns.question.name > Visualize
- Faire donut ou camembert 
- Nombre valeurs 20
- top value / Field dns.question.name.
- Enlever "Group other values"
- event.code : 22

## DNS Answer data
- dns.answer.data
- same 

## Visualize Library 
- agregaded base > Data table > choisir index
- add buckets Split rows > Terms > Filed event.code > include : 22 > Custom label : Event ID
- Autre champs : Terms ... > Field dns.question... Size 30.. 
- Autre champs : Terms ... > Field dns.answer... Size 30.. 
- Add process :Field process.executable
- Add process.name
- Puis save visualization et add au dashboard
![image](https://github.com/user-attachments/assets/044b7369-c858-469e-a669-57626ab333dc)
