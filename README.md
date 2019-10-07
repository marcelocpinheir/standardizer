#"Padronizador" e "Conversor" de planilhas estruturadas

Esta aplica��o converte planilhas estruturadas em arquivos de saida modificados para que sejam importados em sistemas de maneira autom�tica, evitando a necessidade de retrabalho manual ou readequa��es de c�digo.

Para este objetivo criamos uma configura��o de parseamento, para cada tipo de planilha a converter, � poss�vel configurar multiplas padr�es de parseamento, e a ideia � suportar virtualmente qualquer tipo de planilha existente.

O parseamento de planilhas � baseado em padr�es, ou seja, para que seja poss�vel converter uma planilha autom�ticamente, � preciso identificar um padr�o v�lido de dados na mesma, e criar uma configura��o de extra��o (parser) baseada neste padr�o. 

Talvez no futuro tenhamos uma IA que fa�a isso, quem sabe...

##Instala��o
TODO

##Configura��o

Os arquivos de configura��o s�o o ponto principal para que o programa funcione, eles s�o encontrados da pasta 'config' do projeto, quando voc� fizer o clone do projeto haver� 2 arquivos com o sulfixo .example.php para utilizalos.

IMPORTANTE: Renomeie os 2 arquivos para global.php e parser.php respectivamente.

###Arquivo global.php
Cont�m algumas configura��es gerais do projeto e da sa�da gerada, abaixo listadas:

```
<?php

$config = [

    // Defines the output folder for processed files
    'output_folder' => 'output/',
    
    // Temporary folder for pr� conversion output
    'temp_folder' => 'temp/',

    // Defines the output delimiter for csv file
    'delimiter' => ',',

    // Only xls and xlsx are supported by now
    'supported_extensions' => ['xls', 'xlsx'],

    // Only csv is supported by now
    'output_type' => 'csv',
];
```
    
###Arquivo parser.php
Cont�m as configura��es de convers�o poss�veis dentro do projeto, podemos gerar in�meros tipos de saida atrav�s destas configura��es, � importante entender cada uma delas para que voc� consiga ter versatilidade quando estiver construindo o seu conversor, abaixo a lista de configura��es deste arquivo:

```
<?php

$config = [
    /**
     * Define your custom parser name
     */
    'my_parser_name' => [
        /**
         * Discard this number of lines from top of file
         */  
        'discard_top' => 0, // Default 0 dont discart any top lines
        /**
         * Discard this number of lines from the end of file
         */
        'discard_bottom' => 0, // Default 0 dont discard any bottom lines
        /**
         * The number of lines to sumarize before start parsing the content
         * OBS: If the line count not match a divisor, las lines will be ignored
         */
        'line_counter' => 1, // Default 1 parse input file line by line
        /**
         * Discard every line that contains this values
         */
        'discard_contains' => [
            // Values to discard when found
        ],
        /**
         * When this string was found, the file processing ends and the current line is discarded
         */ 
        'end_file_string' => '', // Default empty
        /**
         * Mapper configuration rules
         * See more in: http://todo
         */
        'mapper' => [
            // Implement your mappers
        ],
        /**
         * Customized function definition
         */
        'custom_steps' => [
            // Implement your custom function steps
        ]
    ]
];
```

OBS: Neste arquivo voc� pode definir quantas configura��es de parseamento precisar, recomendamos
tamb�m que salve estas configura��es em local seguro caso precise utilizar novamente no futuro!
    
##Utiliza��o
    
Para executar o conversor basta chamar o arquivo run.php da seguinte maneira:

php run.php -p my_parser_name -f path/to/inputfile.xls

Onde -p representa um padr�o de parseamento configurado no arquivo config/parser.php .
O par�metro -f representa o arquivo a ser processado .

O script suporta tamb�m o comando -h, que ir� mostrar esta documenta��o.

##Documenta��o
    
####Mappers
Um mapeador � uma string estruturada que define, a posi��o do valor a mapear e
as etapas de mapeamento a serem executadas (steps)
    
####Steps
S�o etapas de mapeamento, que geram uma saida para o campo definido, e s�o executadas
sequencialmente, sendo sua saida injetada no proximo step e finalmente no arquivo de output.
Cada step aceita seus par�metros especificos, veja a documenta��o para exemplos.

####Configurando um Mapper
Sintaxe:
```
'position:1,"A";step1:"val1";step2:"valor1","valX"'
```

IMPORTANTE:
O primeiro step deve informar a posi��o do valor desejado (step 'pos').
Os mapeador deve ser definido entre aspas simples ''.
Valores de steps do tipo string s�o definidos entre aspas duplas "".
Cada novo step adicionado recebe o valor de saida do ultimo step.

####Lista de Steps Dispon�veis
**position step** - Posi��o do valor na planilha.

Sintaxe:
```
'position:"[column_letter]",[Opcional: row_counter_index]'
```
Exemplo: 
Retorna o valor no indice de linha 1 coluna A para a saida. 
Entrada: 'texto do campo'
```
    'campo' => 'position:1,"A"'
```
Saida: 'texto do campo'
IMPORTANTE: O campo index, representa o n�mero da linha contada quando os dados estiverem presentes em multiplas linhas, ele n�o � necess�rio e pode ser omitido, caso o processamento seja linha a linha.
            
**split step** - Separa o valor e retorna uma parte baseado na posi��o.
Sintaxe:
```
'split:"[separator]",[index]'
```
Exemplo: 
Separa o valor na posi��o usando / e retorna o valor na posi��o 3
Entrada: 'campo/separado/por/barras'
```
'campo' => 'position:1,"A";split:"/",3'
```
Saida: 'por'
            
**equals step** - Muda o valor de saida caso o valor atual seja igual a um valor especificado
Sintaxe:
```
'equals:"[valor]","[value_if_true]","[value_if_false]"'
```
Exemplo 1:
Entrada: 'valor_atual'
```
'campo' => 'position:1,"A";equals:"valor_atual","novo_valor"'
```
Saida: 'novo_valor'

Exemplo 2:
Entrada: 'valor_atual'
```
'campo' => 'position:1,"A";equals:"diferente","novo_valor","valor_diferente"'
```
Saida: 'valor_diferente'

Observa��es:
Se o *value_if_false* n�o for informado, a fun��o ir� retornar vazio.
Se o *value_if_true* esteja vazio ou em branco, a fun��o ir� retornar vazio.
A palavra chave SELF (sem aspas) retorna o valor atual
A palavra chave PREV (sem aspas) retorna o valor encontrado o mesmo field da linha anterior
            
**numbers step** - Filtra o valor do campo deixando s�mente os n�meros
Sintaxe: 
```
'numbers'
```
Exemplo:
Entrada: '1a2b3c'
```
'campo' => 'position:1,"A";numbers'
```
Saida: '123'
            
**replace step** - Substitui todas as ocorrencias no valor
Sintaxe: 'replace:[valor_original],[novo_valor]'
Exemplo:
Entrada: '555'
```
'campo' => 'position:1,"A";replace:"5","N"'
```
Saida: 'NNN'
        
**custom step** - Executa uma fun��o de step customizada
Sintaxe: 'custom:customFunction,[Opcional $param1],[Opcional $param2]'
Exemplo:
Entrada: '(19)999999999  (21)33333333'
```
'campo' => 'position:1,"A";custom:customPhoneParser'
```
Saida: '19999999999;2133333333'


###Criando steps customizados
As vezes � preciso criar um step customizado para aplicar em um determinado campo, ou v�rios campos.
Para fazer isso utilize o campo 'custom_steps' no arquivo de configura��es da seguinte forma:
```
...
'custom_steps' => [
    'customFunction' => function ($string) {
        $phones = explode('  ', $string);
        foreach($phones as $key => $phone) {
            // Fa�a seu c�digo
            // Returne uma string
        }
    },
],
...
```
Exemplo: 
Fun��o utilizada como no exemplo acima
```
...
'custom_steps' => [
    /**
        * Executa um parse customizado para um campo de telefone
        */
    'customPhoneParser' => function ($string) {
        $phones = explode('  ', $string);
        foreach($phones as $key => $phone) {
            $phoneCleaned = preg_replace('/[^0-9]/', '', $phone);
            if ($phoneCleaned == '') {
                unset($phones[$key]);
            } else {
                $phones[$key] = $phoneCleaned;
            }
            return implode(";", $phones);
        }
    },
],
...
```
IMPORTANTE:
O par�metro principal da fun��o ser� a string contendo o valor atual da posi��o corrente.
O segundo par�metro � um array contendo todos os par�metros adicionais informados na configura��o do mapper.
� poss�vel passar N numero de par�metros para a fun��o customizada.
A fun��o customizada deve retornar um valor do tipo string.