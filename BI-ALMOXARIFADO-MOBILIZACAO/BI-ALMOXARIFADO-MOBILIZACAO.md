# Objetivo
Analisar as movimentações do almoxarifado facilitando o gerenciamento dos itens.  
Obs.: Esse relatório trata apenas de EPIs por movimentação destino (Colaborador) e unidade destino.

# Fonte de Dados
__Banco de Dados:__ MariaDB  
Views:
* almoxarifado_movimentacoes  
* pessoas  
* epiepc_obrigatorios  

# Agendamento de Atualização
Fuso Horário (UTC-03:00) Brasília  
Frequência das Atualizações: Semanal (Segunda a Sexta)  
Hora de Atualização: 10:00 AM

# Processo de ETL (Extração, Transformação e Carregamento)
__View almoxarifado_movimentacoes:__  
Código M dos procedimentos no Power Query:  
> let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_almoxarifado_movimentacoes_View = dolpenge_views_Database{[Name="view_power_bi_almoxarifado_movimentacoes",Kind="View"]}[Data],  
    #"Linhas Filtradas" = Table.SelectRows(view_power_bi_almoxarifado_movimentacoes_View, each ([grupo] = "EPI") and ([unidadeDestino]  
    <> "CUIABÁ - MT" and [unidadeDestino] <> "PALMAS - TO" and [unidadeDestino] <> "VÁRZEA GRANDE - MT")),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Linhas Filtradas", each [dt_conc] >= #datetime(2024, 12, 2, 0, 0, 0)),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Linhas Filtradas1", each Text.Contains([origem], "MOBILIZAÇÃO")),  
    #"Linhas Filtradas3" = Table.SelectRows(#"Linhas Filtradas2", each ([dec_destino] = "PESSOA") and ([cancelado] = "Ativo")),  
    #"Consultas Mescladas" = Table.NestedJoin(#"Linhas Filtradas3", {"id_destino"}, PESSOAS, {"ID"}, "PESSOAS", JoinKind.Inner),  
    #"PESSOAS Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas", "PESSOAS", {"ID_Função Especifica", "Funcao Especifica",  
    "Situação"}, {"ID_Função Especifica", "Funcao Especifica", "Situação"}),  
    #"Colunas Removidas" = Table.RemoveColumns(#"PESSOAS Expandido",{"idmov", "idmovitem", "qtde", "grupo", "unidadeOrigem", "id_tipo_origem",  
    "dec_origem", "id_origem", "dec_destino", "id_tipo_destino"}),  
    #"Colunas Renomeadas" = Table.RenameColumns(#"Colunas Removidas",{{"dt_conc", "Data Conclusão"}, {"dt_acc", "Data Aceite"},  
    {"tipo_de_manifesto", "Tipo de Manifesto"}, {"origem", "Origem"}, {"destino", "Destino"}, {"id_sankhya", "ID_Item"},  
    {"descricao", "Item"}, {"unidadeDestino", "Unidade Destino"}, {"concluido", "Status Movimentação"}, {"cancelado", "Status Registro"},  
    {"aceito", "Assinatura"}, {"produtos_desc_geral", "Item Geral"}, {"id_produtos_desc_geral", "ID_Item Geral"},  
    {"id_destino", "ID_Destino"}}),  
    #"Valor Substituído" = Table.ReplaceValue(#"Colunas Renomeadas","Aceito","Assinado",Replacer.ReplaceText,{"Assinatura"})  
in  
    #"Valor Substituído"  

__Explicação dos procedimentos:__  
1. Conexão com a Fonte de Dados  
* O código se conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Acessa o banco de dados chamado dolpenge_views.  
* Obtém os dados da view chamada view_power_bi_almoxarifado_movimentacoes, que provavelmente já contém dados processados para análise no Power BI.  

2. __Filtro de Linhas__  
* Mantém apenas os registros onde grupo é "EPI" (Equipamentos de Proteção Individual).  
* Exclui registros das unidades "CUIABÁ - MT", "PALMAS - TO" e "VÁRZEA GRANDE - MT".  
* Filtra os dados com base na data:  
* Mantém somente registros com data conclusão (coluna dt_conc) maior ou igual a 2 de dezembro de 2024.
* Filtra registros em que a coluna origem contém a palavra "MOBILIZAÇÃO".
* Filtra novamente os dados:  
* Mantém apenas as movimentações onde dec_destino é "PESSOA".  
* Mantém apenas registros onde cancelado está "Ativo".

3. __Junção com a Tabela de Pessoas__  
* Faz uma junção (JOIN) com a tabela PESSOAS  
* Usa a coluna id_destino da tabela principal e a coluna ID da tabela PESSOAS.  
* Apenas registros correspondentes em ambas as tabelas são mantidos (JoinKind.Inner).  
* Expande as colunas da tabela PESSOAS para adicionar informações sobre a função da pessoa e sua situação.

4. __Removendo Colunas Desnecessárias__  
* Remove colunas irrelevantes para a análise final.

5. __Renomeando Colunas__  
* Renomeia várias colunas para nomes mais intuitivos.

6. __Substituição de Valores__  
* Substitui valores na coluna Assinatura:  
* Se houver "Aceito", ele será alterado para "Assinado".

7. __Retorno Final__  
Retorna a tabela final já transformada!

__View pessoas:__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_pessoas_View = dolpenge_views_Database{[Name="view_power_bi_pessoas",Kind="View"]}[Data],  
    #"Colunas Removidas" = Table.RemoveColumns(view_power_bi_pessoas_View,{"cpf", "sexo", "dt_nascimento",  
    "escolaridade", "status", "idade", "dt_demissao", "cnh", "cnh_validade", "matricula_Dolp", "matricula_Enel", "num_cel_1",  
    "departamento", "cipa", "pcd", "tipo_deficiencia", "tipo", "empresa", "tipo_funcao", "biometria", "login", "id_superior",  
    "superior_imediato", "idtb_empresa_reg"}),  
    #"Colunas Renomeadas" = Table.RenameColumns(#"Colunas Removidas",{{"idtb_oper_pessoa", "ID"}, {"nome", "Colaborador"},  
    {"dt_admissao", "Data Admissão"}, {"funcao_geral", "Função Geral"}, {"situacao", "Situação"}, {"base", "Unidade"}, {"id_funcao",  
    "ID_Função Especifica"}, {"funcao_especifica", "Funcao Especifica"}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Colunas Renomeadas", each ([Funcao Especifica] = "ELETRICISTA DE LINHA VIVA" or  
    [Funcao Especifica] = "ELETRICISTA DE REDES DE DISTRIBUIÇÃO I" or [Funcao Especifica] = "ELETRICISTA DE REDES DE DISTRIBUIÇÃO II"  
    or [Funcao Especifica] = "ELETRICISTA LINHA VIVA" or [Funcao Especifica] = "ELETRICISTA LINHA VIVA ITINERANTE" or [Funcao Especifica]  
    = "ELETRICISTA MONTADOR" or [Funcao Especifica] = "ENCARREGADO" or [Funcao Especifica] = "ENCARREGADO ITINERANTE" or [Funcao Especifica]  
    = "ENCARREGADO LINHA VIVA" or [Funcao Especifica] = "ENCARREGADO LINHA VIVA ITINERANTE" or [Funcao Especifica] = "INSTALADOR ELETRICO ITINERANTE"  
    or [Funcao Especifica] = "INSTALADOR ELÉTRICO CAT A" or [Funcao Especifica] = "INSTALADOR ELÉTRICO CAT B") and ([Unidade] <>  
    "CUIABÁ - MT" and [Unidade] <> "PALMAS - TO" and [Unidade] <> "RIO VERDE - GO")),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Linhas Filtradas", each [Data Admissão] >= #date(2024, 12, 2)),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Linhas Filtradas1", each ([Situação] = "Em Atividade"))  
in  
    #"Linhas Filtradas2"

Explicação dos procedimentos:  
