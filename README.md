# datatable
Tutorial de implementação DataTable com classic ASP

##### 1º passo: atualizar/instalar o componente [DataTable](https://datatables.net/)
##### 2º passo: Adicionar no arquivo que irá consumir o Json - dentro da tag ````<head> ````
```` html
<link rel="stylesheet" type="text/css" href="./datatables/datatables.min.css"> 
<script type="text/javascript" src="./datatables/datatables.min.js"></script>
<script LANGUAGE="JavaScript">

  $(document).ready(function() {
                $('#tableArquivos').DataTable(
                    {
                    "processing": true,
                    "serverSide": true, // ideal para conjunto de dados muito grandes, será feito uma chamada via ajax para cada página
                    "searching": true, //define se haverá filtro(pesquisa) no datatable
                    "ajax": "inserirPaginaQueGeraOJSON.asp", //arquivo no qual será gerado o Json
                    "searchDelay": 3000,
                    "ordering": false,
                    "language" : { //tradução dos componetes que serão exibidos na tabela
                        "lengthMenu": "Mostrar _MENU_ registros por pagina",
                        "zeroRecords": "Nenhum registro encontrado",
                        "info": "Mostrando pag _PAGE_ de _PAGES_",
                        "infoEmpty": "Nenhum registro",
                        "search": "Nome/MASP",
                        "infoFiltered": "(filtrado de um total de _MAX_ total registros)",
                        "paginate": {
                            "next": "Proximo",
                            "previous": "Anterior",
                            "first": "Primeiro",
                            "last": "Ultimo"
                        }
                    },
                    "lengthMenu": [ [50, 100], [50, 100] ]
                });   
                      
        });

 </script>
````
##### Obs.: ao escrever a tabela que receberá  os dados JSON - colocar as colunas respectivamente na ordem dos dados gerados | Informações detalhadas sobre os parâmetros nativos do componente encontram-se no [Manual](https://datatables.net/manual/) do DataTable

##### 3º passo: montar o arquivo onde será gerado o JSON 
``` asp
<%
        limit = ""
        filtro = ""       
        start = ""
        length = ""
        filtro = false 
   i = 0
if request("start") <> "" and request("length") <> ""  then
    start = Clng(request("start"))
    length =  Clng(request("length"))
   
END IF
Dim search
search=Request.QueryString("search[value]")
%>
```
##### Obs.: no meu caso, a versão do Oracle utilizada não comporta o uso de LIMIT e OFFSET que são parâmetros também passados pelo datatable - então foi feita consulta sql com a seguinte estrutura: 
```` asp
<% 
SQLQuery = "SELECT * FROM (SELECT row_number() OVER (ORDER BY a.DATA desc) linha, [...]) WHERE linha between " & start & " AND " & start + length - 1 & fitro   
'A variável filtro recebe o que foi digitado no campo SEARCH. ex.: filtro = "and nome like upper('%" &search&"%')"
'Start e length também são parâmetros enviados pelo datatable
%>
````
##### São os seguintes parâmetros que devem ser retornados no JSON : draw, recordsTotal, recordsFiltered, e data
##### *eu achei mais fácil montar o json manualmente - neste caso eram poucas informações para retornar 
```` asp
<%
if request.QueryString("draw")> 0 then
draw = """" & request.QueryString("draw") & """"
else
draw = """" & 0 & """"
end if 
recordsTotal = """" & total  & """" 'este total é o contador de registros no banco de dados
recordsFiltered = """" & totalRegistros & """" 'totalRegistros = Clng(total("total")) - 1
%>
````
##### E o parâmetro 'data' será preenchido com os dados buscados no seu sql. ex.:
```` asp
<%
 data = ""
            Do While not resultado.EOF  'resultado = Conn.Execute(SQLQuery)
             if i <> 0 then
                   data = data & "," 
                   end if			      
                  id = "[""<b>"&resultado("ID")&" </b>""" &","
                  nome = """<b>" &resultado("NOME")  &" </b>""" & ","
                  dtCampo = """<b>"& resultado("DATA") & "</b>""" & ","                    
                  data = data & id & nome & dtCampo & "]"     
           loop
%>
````
##### 4º passo: Depois é só retornar o Json, formatado da maneira [correta](https://json.org/example.html) 
```` asp
<%
response.ContentType = "application/json"
response.Write("{ ""draw"": "& draw & ",""recordsTotal"": " & recordsTotal & ",""recordsFiltered"": " & recordsFiltered &",""data"":[" &  data  &"]}")
%>
````
