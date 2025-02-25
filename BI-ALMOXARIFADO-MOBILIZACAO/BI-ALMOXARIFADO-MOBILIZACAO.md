# Objetivo do Relatório
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

# Processo de ETL (Extração, Transformação e Carregamento) no Power Query
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
    = "ENCARREGADO LINHA VIVA" or [Funcao Especifica] = "ENCARREGADO LINHA VIVA ITINERANTE" or [Funcao Especifica] =
    "INSTALADOR ELETRICO ITINERANTE"  
    or [Funcao Especifica] = "INSTALADOR ELÉTRICO CAT A" or [Funcao Especifica] = "INSTALADOR ELÉTRICO CAT B") and ([Unidade] <>  
    "CUIABÁ - MT" and [Unidade] <> "PALMAS - TO" and [Unidade] <> "RIO VERDE - GO")),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Linhas Filtradas", each [Data Admissão] >= #date(2024, 12, 2)),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Linhas Filtradas1", each ([Situação] = "Em Atividade"))  
in  
    #"Linhas Filtradas2"

__Explicação dos procedimentos:__  
1. __Conexão com a Fonte de Dados__  
* Conecta-se ao banco de dados MariaDB no servidor "sgddolp.com".  
* Acessa o banco de dados chamado dolpenge_views.  
* Obtém os dados da view view_power_bi_pessoas, que provavelmente contém informações sobre colaboradores.

2. __Remoção de Colunas Desnecessárias__  
* Aqui, são removidas várias colunas que não são necessárias para a análise.  
Exemplo: cpf, sexo, idade, departamento, empresa, cnh, etc.  
Isso reduz o tamanho da tabela e melhora a performance do relatório.

3. __Renomeação de Colunas__  
Renomeia colunas para facilitar o entendimento.  
Exemplo:  
idtb_oper_pessoa → "ID"  
nome → "Colaborador"  
dt_admissao → "Data Admissão"  
funcao_geral → "Função Geral"  
situacao → "Situação"  
base → "Unidade"  
* Isso melhora a legibilidade da tabela no Power BI.

4. __Filtragem__
Mantém apenas os colaboradores que possuem uma das funções específicas listadas.  
Exemplo:  
* "ELETRICISTA DE LINHA VIVA"  
* "ENCARREGADO"  
* "INSTALADOR ELÉTRICO CAT A"  
Além disso, exclui registros de colaboradores das unidades:  
* CUIABÁ - MT  
* PALMAS - TO

* Mantém apenas colaboradores admitidos a partir de 2 de dezembro de 2024.  
* Mantém apenas colaboradores que estão "Em Atividade".  

5. __Retorno da Tabela Transformada__  
* Retorna a tabela final pronta para uso no Power BI.

__View epiepc_obrigatorios:__  
Código M dos procedimentos no Power Query:  
> let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_epiepc_obrigatorios_View = dolpenge_views_Database{[Name="view_power_bi_epiepc_obrigatorios",Kind="View"]}[Data],  
    #"Tipo Alterado" = Table.TransformColumnTypes(view_power_bi_epiepc_obrigatorios_View,{{"qntd", Int64.Type},  
    {"qntdExperiencia", Int64.Type}}),  
    #"Colunas Renomeadas" = Table.RenameColumns(#"Tipo Alterado",{{"idtb_oper_funcao_epiepc", "ID_Função Item"},  
    {"idtb_oper_funcao", "ID_Função Especifica"}, {"idtb_produtos_enel_geral", "ID_Item Geral"}, {"qntd", "Quantidade"},  
    {"qntdExperiencia", "Quantidade Experiencia"}, {"funcao_geral", "Função"}, {"descricao_geral", "Item Geral"}, {"grupo", "Grupo"}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Colunas Renomeadas", each ([Função] = "ELETRICISTA DE LINHA VIVA" or [Função]  
    = "ELETRICISTA DE REDES DE DISTRIBUIÇÃO" or [Função] = "ELETRICISTA LINHA VIVA" or [Função] = "ELETRICISTA LINHA VIVA ITINERANTE"  
    or [Função] = "ELETRICISTA MONTADOR" or [Função] = "ENCARREGADO" or [Função] = "ENCARREGADO ITINERANTE" or [Função]  
    = "ENCARREGADO LINHA VIVA" or [Função] = "ENCARREGADO LINHA VIVA ITINERANTE" or [Função] = "INSTALADOR ELÉTRICO" or [Função]  
    = "INSTALADOR ITINERANTE"))  
in  
    #"Linhas Filtradas"

__Explicação dos procedimentos:__  
1. __Conexão com o Banco de Dados__  
* Estabelece conexão com o servidor sgddolp.com (um banco MariaDB).

2. __Acessando a View do Banco__  
* Acessa o banco de dados dolpenge_views.  
* Obtém os dados da view view_power_bi_almoxarifado_movimentacoes.

3. __Filtragem__  
Filtra apenas os registros onde grupo é "EPI" (Equipamentos de Proteção Individual).  
Exclui movimentações com destino para três unidades específicas:  
* Cuiabá - MT  
* Palmas - TO  
* Várzea Grande - MT

* Mantém apenas registros com data de conclusão (dt_conc) a partir de 02/12/2024.  
* Mantém apenas registros onde a coluna origem contém "MOBILIZAÇÃO".  
Filtra os registros onde:  
* O destino (dec_destino) é "PESSOA".  
* O status do registro (cancelado) está como "Ativo


4. __Junção com Tabela de Pessoas__  
Faz um JOIN (junção) com a tabela PESSOAS, associando:  
* id_destino da tabela principal  
* ID da tabela de pessoas  
Usa JoinKind.Inner, ou seja, mantém apenas registros que existem em ambas as tabelas.  
Expande as colunas da tabela de pessoas:  
* ID_Função Especifica  
* Funcao Especifica  
* Situação

5. __Remoção de Colunas Desnecessárias__  
* Remove colunas que não são mais necessárias na análise

6. __Renomeação de Colunas__  
* Renomeia colunas para facilitar a leitura dos dados.

7. __Substituição de Valores__  
* Substitui o valor "Aceito" por "Assinado" na coluna "Assinatura".

8. __Finalizando a Consulta__  
* Define a tabela final processada como saída da consulta.

__Criação de Tabela em M__  
__Tabela "PESSOAS/ALMOXARIFADO"__  
* Objetivo  
Unificar com tabelas que contém dados divergentes.

Criada no Power Query, removendo algumas colunas e retornando apenas as colunas:  
* ancora  
* base  
* Colaborador  
* Data Admissão  
* função  
* funcao_especifica  
* ID  
* id_funcao
* Situação

# Criação de Tabelas em DAX  
__Tabela ".Pessoas_Almoxarifado"__  
* Objetivo  
Unificar os dados de duas fontes (pessoas e almoxarifado) em uma tabela única, com colunas padronizadas,  
para facilitar análise e relatórios no Power BI.

Codigo da tabela:  
>.Pessoas_Almoxarifado =  
UNION(  
    SELECTCOLUMNS(  
        FILTER(  
            'PESSOAS/ALMOXARIFADO',  
            NOT 'PESSOAS/ALMOXARIFADO'[ID] IN DISTINCT('ALMOXARIFADO'[ID_Destino])  
        ),  
        "Colaborador", 'PESSOAS/ALMOXARIFADO'[Colaborador],  
        "ID_Colaborador", 'PESSOAS/ALMOXARIFADO'[ID],  
        "Função", 'PESSOAS/ALMOXARIFADO'[funcao_especifica],  
        "ID_Função Especifica", 'PESSOAS/ALMOXARIFADO'[id_funcao],  
        "Status", "Pessoas",  
        "Status da Movimentação", "Não Existe Movimentação",  
        "Status do Registro", "Aguardando",  
        "Assinatura", BLANK(),  
        ".StatusItem", "Sem Movimentação",  
        "Unidade de Destino", 'PESSOAS/ALMOXARIFADO'[base],  
        "Destino", BLANK(),  
        "Item", BLANK(),  
        "ID_Item", 0,  
        "ID_Item Geral", BLANK(),  
        "Item Geral", BLANK(),  
        "Data da Movimentação", BLANK()  
    ),  
    SELECTCOLUMNS(  
        'ALMOXARIFADO',  
        "Colaborador", 'ALMOXARIFADO'[DESTINO],  
        "ID_Colaborador", 'ALMOXARIFADO'[ID_Destino],  
        "Função", 'ALMOXARIFADO'[Funcao Especifica],  
        "ID_Função Especifica", 'ALMOXARIFADO'[ID_Função Especifica],  
        "Status", "Almoxarifado",  
        "Status da Movimentação", 'ALMOXARIFADO'[Status Movimentação],  
        "Status do Registro", 'ALMOXARIFADO'[Status Registro],  
        "Assinatura", 'ALMOXARIFADO'[Assinatura],  
        ".StatusItem", 'ALMOXARIFADO'[.StatusItem],  
        "Unidade de Destino", 'ALMOXARIFADO'[Unidade Destino],  
        "Destino", 'ALMOXARIFADO'[Destino],  
        "Item", 'ALMOXARIFADO'[Item],  
        "ID_Item", 'ALMOXARIFADO'[ID_Item],  
        "ID_Item Geral", 'ALMOXARIFADO'[ID_Item Geral],  
        "Item Geral", 'ALMOXARIFADO'[Item Geral],  
        "Data da Movimentação", 'ALMOXARIFADO'[Data Conclusão]  
    )  
)

__Explicação do Código__  
1. __Union__  
A função UNION é usada para combinar duas tabelas, concatenando suas linhas. Ou seja, ela junta os dados da tabela de pessoas  
(que não têm movimentação) com os dados da tabela de almoxarifado (que possuem movimentação).

2. __SELECTCOLUMNS (para PESSOAS/ALMOXARIFADO)__  
* FILTER: Filtra os registros da tabela 'PESSOAS/ALMOXARIFADO' onde o ID da pessoa não está presente na tabela ALMOXARIFADO[ID_Destino].  
Ou seja, ele pega as pessoas que não têm movimentação registrada no almoxarifado.  
* SELECTCOLUMNS: Após o filtro, seleciona as colunas relevantes e renomeia-as para um formato mais amigável.  
Além disso, algumas colunas são preenchidas com valores fixos, como:  
    * "Status" é definido como "Pessoas", pois esses registros são de pessoas sem movimentação.  
    * "Status da Movimentação" é "Não Existe Movimentação".  
    * "Status do Registro" é "Aguardando".  
    * O campo "Assinatura" é em branco (BLANK()), já que não há movimentação para assinar.  
    * Outros campos, como "Item" e "Data da Movimentação", também são preenchidos com valores vazios ou padrão.

3. __SELECTCOLUMNS (para ALMOXARIFADO)__  
* Esta parte do código seleciona registros diretamente da tabela ALMOXARIFADO.  
* Para cada registro, a função SELECTCOLUMNS escolhe as colunas e as renomeia conforme necessário:  
    * "Status" é definido como "Almoxarifado", porque esses registros correspondem a movimentações no almoxarifado.  
    * "Status da Movimentação" é preenchido com os valores da coluna Status Movimentação.  
    * "Assinatura", "Unidade de Destino", "Destino", entre outras, são copiadas diretamente da tabela ALMOXARIFADO.

__Tabela ".TabelaBaseFunção"__  
* Objetivo  
Essa tabela pode ser útil para análises relacionadas ao controle de itens, especialmente quando se trata de EPI  
(Equipamento de Proteção Individual), como a "Capa de Chuva", agrupando os dados por função e contando os itens de forma eficiente.

Código da Tabela:  
>.TabelaBaseFunção =  
SUMMARIZE(  
    'EPIEPC_OBRIGATORIO',  
    'EPIEPC_OBRIGATORIO'[ID_Função Especifica],  
    "Item Geral Agrupado",  
        MAXX(  
            FILTER(  
                'EPIEPC_OBRIGATORIO',  
                CONTAINSSTRING('EPIEPC_OBRIGATORIO'[Item Geral], "Capa de Chuva")  
            ),  
            "Capa de Chuva"  
        ),  
    "QuantidadeItens",  
        CALCULATE(  
            DISTINCTCOUNT('EPIEPC_OBRIGATORIO'[Item Geral]),  
            ALLEXCEPT(  
                'EPIEPC_OBRIGATORIO',  
                'EPIEPC_OBRIGATORIO'[ID_Função Especifica]  
            )  
        )  
)

__Explicação do Código__  
1. __SUMMARIZE__  
A função SUMMARIZE cria uma tabela de resumo, com base em uma ou mais colunas de agrupamento e cálculos adicionais.  
No caso, a tabela resultante será agrupada pela coluna 'EPIEPC_OBRIGATORIO'[ID_Função Especifica] e terá duas colunas adicionais:  
"Item Geral Agrupado" e "QuantidadeItens".

2. __"Item Geral Agrupado" (Cálculo MAXX + FILTER + CONTAINSSTRING)__  
* MAXX: Esta função retorna o maior valor de uma expressão. Porém, aqui ela está sendo usada para retornar um valor fixo  
("Capa de Chuva") para as linhas que atendem à condição no FILTER.  
* FILTER: Filtra a tabela 'EPIEPC_OBRIGATORIO' para que apenas as linhas onde o campo 'Item Geral' contém a string  
"Capa de Chuva" sejam consideradas.  
* CONTAINSSTRING: Verifica se a string "Capa de Chuva" está contida no campo 'Item Geral'.  
* Portanto, a coluna "Item Geral Agrupado" vai ter o valor "Capa de Chuva" para todos os agrupamentos em que o item geral  
contiver essa string. Para os outros agrupamentos, o valor será BLANK().  

3. __"QuantidadeItens" (Cálculo DISTINCTCOUNT + ALLEXCEPT)__  
* DISTINCTCOUNT: Conta o número distinto de valores na coluna 'Item Geral'.  
* ALLEXCEPT: Remove todos os filtros da tabela 'EPIEPC_OBRIGATORIO', exceto o filtro na coluna 'ID_Função Especifica'.  
Isso significa que o cálculo de DISTINCTCOUNT será feito para todos os itens, mas agrupado por ID_Função Especifica.  
* CALCULATE: Permite alterar o contexto de cálculo, aplicando o filtro ALLEXCEPT, o que faz com que a contagem de itens seja  
feita dentro de cada ID_Função Especifica, mas sem se preocupar com outros filtros que possam existir na tabela.  
A coluna "QuantidadeItens" retornará o número distinto de itens da coluna 'Item Geral' para cada ID_Função Especifica,  
considerando todas as linhas da tabela que compartilham o mesmo valor em ID_Função Especifica.

# Colunas Calculadas e Medidas
__Coluna ".StatusItem":__  
.StatusItem = IF(  
    ALMOXARIFADO[Status Movimentação] = "Em Aberto",  
    "Item em Movimentação",  
    IF(  
        ALMOXARIFADO[Status Movimentação] = "Concluído" &&  
        ALMOXARIFADO[Status Registro] = "Ativo",  
        IF(  
            ALMOXARIFADO[Assinatura] = "Assinado",  
            "Item Entregue",  
            "Item Não Assinado"  
        ),  
        BLANK()  
    )  
)

__Explicação do Código__  
A coluna calculada ".StatusItem" determina o status de um item na tabela "Almoxarifado" com base em três condições:  
1. Se o status de movimentação for "Em Aberto", o item está "Em Movimentação".  
2. Se o status de movimentação for "Concluído" e o status de registro for "Ativo":  
    * Se o item foi assinado, o status será "Item Entregue".  
    * Se o item não foi assinado, o status será "Item Não Assinado".  
3. Caso nenhuma dessas condições seja atendida, o valor será em branco (BLANK()).

Em resumo, a coluna indica se o item está em movimentação, se foi entregue ou não assinado, ou se não se encaixa em nenhuma  
dessas condições, deixando o status em branco.

__Coluna ".EPI_Obrigatorio":__  
.EPI_Obrigatorio =  
IF(  
    NOT ISBLANK(  
        LOOKUPVALUE(  
            'EPIEPC_OBRIGATORIO'[ID_Item Geral],  
            'EPIEPC_OBRIGATORIO'[ID_Item Geral], '.Pessoas_Almoxarifado'[ID_Item Geral]  
        )  
    ),  
    "EPI Obrigatorio",  
    BLANK()  
)

__Explicação do Código__  
A fórmula DAX da coluna verifica se um item específico é considerado EPI Obrigatório com base em uma  
comparação entre duas tabelas.

A coluna verifica se o ID_Item Geral de um item na tabela '.Pessoas_Almoxarifado' existe na tabela 'EPIEPC_OBRIGATORIO'.  
Se existir, o valor da coluna será "EPI Obrigatorio". Caso contrário, o valor será em branco (BLANK()).  
Ou seja, a coluna indica que um item é EPI Obrigatório caso ele esteja listado na tabela de itens obrigatórios.

__Coluna ".Quant.ItensAlmoxarifado":__  
.Quant.ItensAlmoxarifado =  
IF(  
    ISBLANK(MINX(ALL('.Pessoas_Almoxarifado'[ID_Item Geral]), '.Pessoas_Almoxarifado'[ID_Item Geral])),  
    BLANK(),  
    CALCULATE(  
        DISTINCTCOUNT('.Pessoas_Almoxarifado'[ID_Item Geral])  
        + COUNTROWS(  
            FILTER(  
                '.Pessoas_Almoxarifado',  
                '.Pessoas_Almoxarifado'[Item Geral] = "Capa de Chuva"  
            )  
        ) - IF(  
            COUNTROWS(  
                FILTER(  
                    '.Pessoas_Almoxarifado',  
                    '.Pessoas_Almoxarifado'[Item Geral] = "Capa de Chuva"  
                )  
            ) > 0, 1, 0  
        ),  
        '.Pessoas_Almoxarifado'[.EPI_Obrigatorio] = "EPI Obrigatorio",  
        ALLEXCEPT(  
            '.Pessoas_Almoxarifado',  
            '.Pessoas_Almoxarifado'[ID_Função Especifica],  
            '.Pessoas_Almoxarifado'[ID_Colaborador]  
        )  
    )  
)

__Explicação do Código__  
A fórmula da coluna calcula a quantidade de itens no almoxarifado para um determinado colaborador e função específica,  
levando em conta algumas condições especiais relacionadas a itens, como "Capa de Chuva" e EPI Obrigatório

A coluna calcula a quantidade de itens únicos no almoxarifado para um determinado colaborador e função, levando em consideração:  
    * A contagem de itens distintos no almoxarifado (baseado no ID_Item Geral).  
    * A adição de itens "Capa de Chuva" ao total.  
    * A subtração de 1 unidade, se houver ao menos uma "Capa de Chuva", para evitar duplicação na contagem.  
    * Apenas os itens que são EPI Obrigatório são considerados.  
    * A contagem é realizada para cada combinção de colaborador e função, considerando apenas esses filtros.  
A fórmula essencialmente retorna a quantidade de itens relacionados a um colaborador e função, com ajustes específicos  
para itens especiais, como "Capa de Chuva" e itens que são EPI Obrigatório.

__Coluna ".Quant.ItensObrigatorios":__  
.Quant.ItensObrigatorios =  
LOOKUPVALUE(  
    '.TabelaBaseFunção'[QuantidadeItens],  
    '.TabelaBaseFunção'[ID_Função Especifica],  
    '.Pessoas_Almoxarifado'[ID_Função Especifica]  
)

__Explicação do Código__  
A coluna utiliza a função LOOKUPVALUE para retornar o valor da quantidade de itens obrigatórios com base na função específica  
associada a cada item na tabela .Pessoas_Almoxarifado.

Retorna a quantidade de itens obrigatórios para a função específica de cada colaborador, utilizando as informações da tabela  
.TabelaBaseFunção. Ela faz isso procurando a quantidade de itens na tabela .TabelaBaseFunção, com base no ID_Função Especifica  
da tabela .Pessoas_Almoxarifado.

__Coluna ".StatusFicha" (Ainda não usada):__
.StatusFicha =  
VAR QuantItensObrigatorios =  
    '.Pessoas_Almoxarifado'[.Quant.ItensObrigatorios]  -- Quantidade de EPIs Obrigatórios  

VAR QuantItensEntregues =  
    '.Pessoas_Almoxarifado'[.Quant.ItensAlmoxarifado]  -- Quantidade de EPIs por Colaborador  

VAR ItensEmMovimentacao =  
    CALCULATE(  
        COUNTROWS('.Pessoas_Almoxarifado'),  
        '.Pessoas_Almoxarifado'[.EPI_Obrigatorio] = "EPI Obrigatorio",  
        '.Pessoas_Almoxarifado'[.StatusItem] = "Item Não Assinado",  
        ALLEXCEPT(  
            '.Pessoas_Almoxarifado',  
            '.Pessoas_Almoxarifado'[ID_Colaborador]  
        )  
    )  

RETURN  
IF(  
    '.Pessoas_Almoxarifado'[.EPI_Obrigatorio] = "EPI Obrigatorio" &&  
    QuantItensEntregues < QuantItensObrigatorios &&  
    ItensEmMovimentacao > 0,  
    "Ficha Incompleta, Aguardando Assinaturas",  
    BLANK()  
)

__Explicação do Código__  
A coluna determina o status da ficha de um colaborador com base na quantidade de EPIs obrigatórios que ele recebeu,  
no número de itens em movimentação e se há itens não assinados. A fórmula segue a lógica abaixo:

Retorna "Ficha Incompleta, Aguardando Assinaturas" se o colaborador tiver menos EPIs entregues do que os EPIs obrigatórios  
e se houver itens em movimentação não assinados. Caso contrário, a coluna retorna em branco.  
Esse status pode ser usado para monitorar se os colaboradores estão com sua ficha de EPIs completa ou se ainda estão  
aguardando a entrega e assinatura de alguns itens obrigatórios.

Obs.: Código incompleto

__Medida "-Colab.SemMovimentação":__  
-Colab.SemMovimentação =  
COALESCE(  
    CALCULATE(  
        DISTINCTCOUNT('.Pessoas_Almoxarifado'[ID_Colaborador]),  
        FILTER(  
            '.Pessoas_Almoxarifado',  
            '.Pessoas_Almoxarifado'[Status da Movimentação] = "Não Existe Movimentação"  
        )  
    ),  
    0  
)

__Explicação do Código__  
Tem o objetivo de calcular o número distinto de colaboradores (representados pelo campo ID_Colaborador da tabela .Pessoas_Almoxarifado)  
que não têm movimentação registrada (onde o campo Status da Movimentação é igual a "Não Existe Movimentação").

__Medida "-EPIsEntregues":__  
-EPIsEntregues =  
COALESCE(  
    COUNTROWS(  
        FILTER(  
            '.Pessoas_Almoxarifado',  
            '.Pessoas_Almoxarifado'[.StatusItem] = "Item Entregue" &&  
            '.Pessoas_Almoxarifado'[Assinatura] = "Assinado"  
        )  
    ),  
    0  
)

__Explicação do Código__  
Tem o objetivo de contar o número de registros de colaboradores (na tabela .Pessoas_Almoxarifado) onde o item foi entregue  
e a assinatura foi registrada como "Assinado". Caso não haja nenhum registro que atenda a esses critérios, ela retorna 0.

__Medida "-EPIsEntreguesNãoAssinados":__  
-EPIsEntreguesNãoAssinados =  
COALESCE(  
    COUNTROWS(  
        FILTER(  
            '.Pessoas_Almoxarifado',  
            '.Pessoas_Almoxarifado'[.StatusItem] = "Item Não Assinado"  
        )  
    ),  
    0  
)

__Explicação do Código__  
Tem o objetivo de contar o número de registros de colaboradores (na tabela .Pessoas_Almoxarifado) onde o item foi entregue,  
mas ainda não foi assinado (indicado pelo status "Item Não Assinado"). Caso não haja nenhum registro que atenda a esse critério, ela retorna 0.

__Medida "-EPIsMovimentaçãoAberta":__  
-EPIsMovimentaçãoAberta =  
COALESCE(  
    COUNTROWS(  
        FILTER(  
            '.Pessoas_Almoxarifado',  
            '.Pessoas_Almoxarifado'[Status da Movimentação] = "Em Aberto"  
        )  
    ),  
    0  
)

__Explicação do Código__  
Tem o objetivo de contar o número de registros de colaboradores (na tabela .Pessoas_Almoxarifado) onde o status da movimentação  
está marcado como "Em Aberto". Caso não haja nenhum registro com esse status, a medida retorna 0.

__Medida "-ItensFaltantesPorColaborador":__  
-ItensFaltantesPorColaborador =  
VAR ItensObrigatorios =  
    CALCULATETABLE(  
        VALUES(EPIEPC_OBRIGATORIO[ID_Item Geral]),  
        NOT ISBLANK(EPIEPC_OBRIGATORIO[ID_Item Geral])  
    )  
VAR ItensEntregues =  
    CALCULATETABLE(  
        VALUES('.Pessoas_Almoxarifado'[ID_Item Geral]),  
        '.Pessoas_Almoxarifado'[ID_Colaborador] = MAX('.Pessoas_Almoxarifado'[ID_Colaborador])  
    )  
VAR ItensFaltantes = EXCEPT(ItensObrigatorios, ItensEntregues)  
RETURN  
    CONCATENATEX(  
        FILTER(  
            EPIEPC_OBRIGATORIO,  
            EPIEPC_OBRIGATORIO[ID_Item Geral] IN ItensFaltantes  
        ),  
        EPIEPC_OBRIGATORIO[Item Geral],  
        ", ",  
        EPIEPC_OBRIGATORIO[Item Geral]  
    )

__Explicação do Código__  
Tem o objetivo de retornar uma lista dos itens obrigatórios que não foram entregues a um colaborador específico.  
A lista é composta pelos itens que estão definidos como obrigatórios na tabela EPIEPC_OBRIGATORIO, mas que não aparecem na tabela  
.Pessoas_Almoxarifado para aquele colaborador.

__Medida "-Quantidade_Itens_Por_Colaborador":__  
-Quantidade_Itens_Por_Colaborador =  
VAR ItensPorColaborador =  
    CALCULATE(  
        COUNT('.Pessoas_Almoxarifado'[Item]),  
        ALLEXCEPT('.Pessoas_Almoxarifado', '.Pessoas_Almoxarifado'[ID_Colaborador], '.Pessoas_Almoxarifado'[Item])  
    )  
RETURN  
    COALESCE(ItensPorColaborador, 0)

__Explicação do Código__  
Tem o objetivo de calcular a quantidade de itens diferentes que foram entregues a cada colaborador na tabela .Pessoas_Almoxarifado,  
levando em consideração o ID_Colaborador e o Item.

# Indicadores KPIs  
O relatório é composto por 3 gráficos (1 grafico de tabela e 2 graficos de pizza) e 3 segmnetadores de dados e 5 cartões

__Gráfico de Tabela:__  
1. KPI: Status de cada item  
2. KPI: Itens por movimentação destino (Colaborador)  
3. KPI: Quantidade de itens por colaborador  
4. KPI: Unidade destino de cada item

__Graficos de Pizza:__  
1. KPI: Quantidade de itens por unidade  
2. KPI: Quantidade de funcionários sem movimentação de estoque por unidade

__Cartões__  
1. KPI: Total de movimentações distintas (Colaboradores)  
2. KPI: Total de colaboradores sem movimentações no estoque  
3. KPI: EPIs com movimentação em aberto  
4. KPI: EPIs entregues  
5. KPI: EPIs entregues e não assinados

__Segmentadores de Dados (Filtros dinâmicos)__  
1. Status do item  
    * Item em Movimentação  
    * Item entregue  
    * Item não assinado  
    * Item sem movimentação de estoque  
2. Unidade de destino  
    * Caldas Novas  
    * Catalão  
    * Itumbiara  
    * Morrinhos  
    * Pires do Rio  
3. Função  
4. Data da movimentaçao do item

teste do teste