# ProjetoBdEletricaPower

DROP DATABASE IF EXISTS marcos_db;

CREATE DATABASE marcos_db;

USE marcos_db;

CREATE TABLE estado(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL 
,sigla CHAR (2) NOT NULL 
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,CONSTRAINT pk_estado PRIMARY KEY (id)
,CONSTRAINT coluna_situaçao_estado CHECK (situaçao IN ('A','I'))
,CONSTRAINT estado_unico UNIQUE (nome,sigla)
);

INSERT INTO estado (nome,sigla) VALUES ('PARANÁ','PR');
INSERT INTO estado (nome,sigla) VALUES ('SÃO PAULO','SP');


SELECT id,nome,sigla,situaçao,data_cadastro FROM estado;

CREATE TABLE cidade(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,estado_id INT NOT NULL
,CONSTRAINT pk_cidade PRIMARY KEY (id)
,CONSTRAINT fk_cidade_estado FOREIGN KEY (estado_id) REFERENCES estado (id)
,CONSTRAINT coluna_situaçao_cidade CHECK (situaçao IN ('A', 'I'))
,CONSTRAINT cidade_unica UNIQUE (nome, estado_id)
);

INSERT INTO cidade (nome,estado_id) VALUES ('PARANAVAÍ',1);
INSERT INTO cidade (nome,estado_id) VALUES ('CURITIBA',1);
INSERT INTO cidade (nome,estado_id) VALUES ('LONDRINA',1);
INSERT INTO cidade (nome,estado_id) VALUES ('CAMPINAS',2);

SELECT *
FROM cidade
INNER JOIN estado ON cidade.estado_id = estado.id; /** Essa consulta ajuda saber de qual estado é determinada cidade. 
A importancia será desse complemento do estado são para novos cadastros, pois podem existir algumas cidades com nomes repetidos em todo o país**/


CREATE TABLE funcionario(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,cpf CHAR (12) NOT NULL
,endereço VARCHAR (200)
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,cidade_id INT NOT NULL
,CONSTRAINT pk_funcionario PRIMARY KEY (id)
,CONSTRAINT fk_funcionario_cidade FOREIGN KEY (cidade_id) REFERENCES cidade (id)
,CONSTRAINT coluna_situçao_funcionario CHECK (situaçao IN ('A','I'))
,CONSTRAINT funcionario_unico UNIQUE (cpf,cidade_id)
);

INSERT INTO funcionario (nome,cpf,endereço,cidade_id) VALUES ('MARCOS',49978367845,'RUA DR SILVIO VIDAL JAD SÃO CRISTOVÃO NUMERO 355',1);
INSERT INTO funcionario (nome,cpf,endereço,cidade_id) VALUES ('ADRIANO',59223261233,'AVENIDA TANCREDO NEVES JAD PRUDÊNCIO NUMERO 945',1);
INSERT INTO funcionario (nome,cpf,endereço,cidade_id) VALUES ('RONALDO',10116445574,'RUA MANOEL RIBAS CENTRO NUMERO 641',1);
INSERT INTO funcionario (nome,cpf,endereço,cidade_id) VALUES ('JOANA',21318836958,'RUA SANTA CATARINA VILA CITY NUMERO 123',1);

DROP FUNCTION pegar_funcionario;
DELIMITER //
CREATE FUNCTION pegar_funcionario( pid INT) 
RETURNS VARCHAR (120) 
DETERMINISTIC
BEGIN
    DECLARE retorno VARCHAR(120);
    DECLARE quantidade INT (1);
    
      SELECT COUNT(*) INTO quantidade FROM funcionario WHERE pid = funcionario_id;
      
        IF quantidade = 1 THEN
			BEGIN
				SELECT nome INTO retorno FROM funcionario WHERE  funcionario_id = pid;
            END;
        ELSE
			BEGIN
				SET retorno = 'funcionario inexistente';
			END;
        END IF;
        RETURN retorno;
END;
//
DELIMITER ;

SELECT pegar_funcionario(3);

DROP FUNCTION funcionariosApos;
DELIMITER //
CREATE FUNCTION funcionariosApos(dt DATETIME)
RETURNS DATETIME
DETERMINISTIC
BEGIN
	DECLARE resultado DATETIME;
	SELECT nome INTO resultado FROM funcionario WHERE data_cadastro >= dt;
RETURN resultado;
END;
//
DELIMITER ;

SELECT *, funcionariosApos('2021-10-01');

CREATE TABLE telefone(
id INT NOT NULL AUTO_INCREMENT
,ddd CHAR (2) NOT NULL
,numero CHAR (20) NOT NULL
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,funcionario_id INT NOT NULL
,CONSTRAINT pk_telefone PRIMARY KEY (id)
,CONSTRAINT fk_telefone_funcionario FOREIGN KEY (funcionario_id) REFERENCES funcionario (id)
,CONSTRAINT coluna_situçao_telefone CHECK (situaçao IN ('A','I'))
,CONSTRAINT telefone_unico UNIQUE (funcionario_id)
);

INSERT INTO telefone (ddd,numero,funcionario_id) VALUES (44,995412297,1);
INSERT INTO telefone (ddd,numero,funcionario_id) VALUES (44,988416977,2);
INSERT INTO telefone (ddd,numero,funcionario_id) VALUES (44,999887023,3);
INSERT INTO telefone (ddd,numero,funcionario_id) VALUES (44,998374416,4);
 
SELECT *
FROM telefone
INNER JOIN funcionario ON telefone.funcionario_id = funcionario.id; /** Essa consulta nos permite relacionar os telefones com todos funcionarios.
A importacia é observar as dastas que foram feitas os cadastros, pois a cada 1 ano a empresa devera realizar a atualização de todos os dados cadastrais, para preenchimento do eSocial **/


CREATE TABLE caixa(
id INT NOT NULL AUTO_INCREMENT
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,saldo DECIMAL (9,2)
,funcionario_id INT NOT NULL
,CONSTRAINT pk_caixa PRIMARY KEY (id)
,CONSTRAINT fk_caixa_funcionario FOREIGN KEY (funcionario_id) REFERENCES funcionario (id)
,CONSTRAINT coluna_situaçao_caixa CHECK (situaçao IN ('A','F'))
,CONSTRAINT caixa_unico UNIQUE (funcionario_id)
);

INSERT INTO caixa (saldo,funcionario_id) VALUES (1400.00,1);
INSERT INTO caixa (saldo,funcionario_id) VALUES (950.00,2);



CREATE TABLE iten_caixa(
id INT NOT NULL AUTO_INCREMENT
,valor DECIMAL (9,2)
,especie VARCHAR (45)
,hora VARCHAR (45)
,caixa_id INT NOT NULL
,CONSTRAINT pk_iten_caixa PRIMARY KEY (id)
,CONSTRAINT fk_iten_caixa_caixa FOREIGN KEY (caixa_id) REFERENCES caixa (id)
,CONSTRAINT iten_caixa_unico UNIQUE (caixa_id)
);

CREATE TABLE cliente(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,endereço VARCHAR (200) NOT NULL
,contato VARCHAR (100) NOT NULL 
,telefone CHAR (20) NOT NULL
,ddd_telefone CHAR (2) NOT NULL 
,cpf_cnpj CHAR (20) NOT NULL 
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,cidade_id INT NOT NULL
,CONSTRAINT pk_cliente PRIMARY KEY (id)
,CONSTRAINT fk_cliente_cidade FOREIGN KEY (cidade_id) REFERENCES cidade (id)
,CONSTRAINT coluna_situaçao_cliente CHECK (situaçao IN ('A','I'))
,CONSTRAINT cliente_unico UNIQUE (contato,cpf_cnpj,cidade_id)
);


INSERT INTO cliente (nome,cpf_cnpj,endereço,contato,telefone,ddd_telefone,cidade_id) VALUES ('JE EMPREENDIMENTOS',69304100000108,'RUA PERNAMBUCO CENTRO NUMERO 362','JE@GMAIL.COM',34226698,44,1);
INSERT INTO cliente (nome,cpf_cnpj,endereço,contato,telefone,ddd_telefone,cidade_id) VALUES ('MA BURITTI',85684280000157,'RUA AMADOR AGUIAR JAD OURO BRANCO NUMERO 455','MABURITTI@HOMTMAIL.COM',998346621,44,1);
INSERT INTO cliente (nome,cpf_cnpj,endereço,contato,telefone,ddd_telefone,cidade_id) VALUES ('JJ CONSTRUÇÕES',29385774000160,'RUA SALVADOR CENTRO NUMERO 1001','JJ@GMAIL.COM',34463366,44,1);
INSERT INTO cliente (nome,cpf_cnpj,endereço,contato,telefone,ddd_telefone,cidade_id) VALUES ('RUBENS',20019587008,'RUA PARA VILA MARIANA NUMERO 741','RUBENS@HOTMAIL.COM',997354142,44,1);
INSERT INTO cliente (nome,cpf_cnpj,endereço,contato,telefone,ddd_telefone,cidade_id) VALUES ('JOAO',30145298755,'RUA DISTRITO FEDERAL CENTRO NUMERO 678','JOAO_PEREIRA@GMAIL.COM',988745246,44,1);
INSERT INTO cliente (nome,cpf_cnpj,endereço,contato,telefone,ddd_telefone,cidade_id) VALUES ('MARIA',69304100000108,'AVENIDA DOMINGOS SANCHES PARQUE MORUMBI NUMERO 851','MARIA@HOTMAIL.COM',34224528,44,1);


CREATE TABLE serviço(
id INT NOT NULL AUTO_INCREMENT
,situaçao CHAR (1) NOT NULL DEFAULT 'C'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,funcionario_id INT NOT NULL
,CONSTRAINT pk_serviço PRIMARY KEY (id)
,CONSTRAINT fk_serviço_funcionario FOREIGN KEY (funcionario_id) REFERENCES funcionario (id)
,CONSTRAINT coluna_situaçao_serviço CHECK (situaçao IN ('C','N')) -- C = Concluido ou N = Não Concluido
);

INSERT INTO serviço (funcionario_id) VALUES (2);
INSERT INTO serviço (funcionario_id) VALUES (2);
INSERT INTO serviço (funcionario_id) VALUES (1);
INSERT INTO serviço (funcionario_id) VALUES (1);
INSERT INTO serviço (funcionario_id) VALUES (2);

SELECT *
FROM serviço 
INNER JOIN funcionario ON serviço.funcionario_id = funcionario.id; /** Essa consulta é para saber a soma dos serviços. 
A importancia de saber desses numeros, é de fazer o equilibrio das demandas entre todos os funcionarios**/



CREATE TABLE ordem_serviço(
id INT NOT NULL AUTO_INCREMENT
,valor DECIMAL (9,2)
,descriçao VARCHAR (200) NOT NULL
,pagamento VARCHAR (10) NOT NULL
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,cliente_id INT NOT NULL
,funcionario_id INT NOT NULL
,CONSTRAINT pk_ordem_serviço PRIMARY KEY (id)
,CONSTRAINT fk_ordem_serviço_cliente FOREIGN KEY (cliente_id) REFERENCES cliente (id)
,CONSTRAINT fk_ordem_serviço_funcionario FOREIGN KEY (funcionario_id) REFERENCES funcionario (id)
,CONSTRAINT ordem_serviço_unico UNIQUE (cliente_id,funcionario_id)
);

INSERT INTO ordem_serviço (valor,descriçao,pagamento,cliente_id,funcionario_id) VALUES (150.00,'Troca e Remanejamento de Tomadas','avista',1,2);
INSERT INTO ordem_serviço (valor,descriçao,pagamento,cliente_id,funcionario_id) VALUES (250.00,'Instalação de Luminárias','avista',2,1);
INSERT INTO ordem_serviço (valor,descriçao,pagamento,cliente_id,funcionario_id) VALUES (500.00,'Troca de Cabeamento Elétrico','parcelado',3,2);
INSERT INTO ordem_serviço (valor,descriçao,pagamento,cliente_id,funcionario_id) VALUES (1000.00,'Instação do Quadro de Distribuição de Energia','avista',4,3);

DROP FUNCTION calcula_comissao;**/
DELIMITER //
CREATE FUNCTION calcula_comissao(valor DECIMAL (9,2))
RETURNS DECIMAL (9,2) DETERMINISTIC
BEGIN
	DECLARE valor_comissao DECIMAL (9,2);
    
	IF valor > 1000  THEN
		BEGIN
			SET valor_comissao = valor * 1.10 - valor;
		END;
	ELSEIF valor > 400  THEN
		BEGIN
			SET valor_comissao = valor * 1.05 - valor;
		END;
	ElSE
		BEGIN
			SET valor_comissao = 0.00;
		END;
		END IF;
	RETURN valor_comissao;
END;
//
DELIMITER ;

# Invocando a função
SELECT *,calcula_comissao(ordem_serviço.valor) AS comissao
FROM ordem_serviço
WHERE pagamento = 'avista';


SELECT *
FROM ordem_serviço 
INNER JOIN cliente ON ordem_serviço.cliente_id = cliente.id
INNER JOIN funcionario ON ordem_serviço.funcionario_id = funcionario.id; /** Essa consulta mostra em quantos serviços cada funcionario estão atuando.
A importancia da consulta, é de ver qual é o responsavel direto pelo serviço, para possiveis retorno aos clientes**/

CREATE TABLE fornecedor(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,cnpj VARCHAR (200) NOT NULL
,endereço VARCHAR (200) NOT NULL
,telefone CHAR (20) NOT NULL
,ddd_telefone CHAR (2) NOT NULL 
,contato CHAR (100) NOT NULL 
,situaçao CHAR (1) NOT NULL DEFAULT 'A'
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,cidade_id INT NOT NULL
,CONSTRAINT pk_fornecedor PRIMARY KEY (id)
,CONSTRAINT fk_fornecedor_cidade FOREIGN KEY (cidade_id) REFERENCES cidade (id)
,CONSTRAINT coluna_situaçao_fornecedor CHECK (situaçao IN ('A','I'))
,CONSTRAINT fornecedor_unico UNIQUE (contato,cnpj,cidade_id)
);

INSERT INTO fornecedor (nome,cnpj,endereço,telefone,ddd_telefone,contato,cidade_id) VALUES ('ENERGIZA LTDA',44828671000110,'RUA MATO GROSSO CENTRO NUMERO 389',34459944,44,'ENERGIZA@HOTMAIL.COM',1);
INSERT INTO fornecedor (nome,cnpj,endereço,telefone,ddd_telefone,contato,cidade_id) VALUES ('POWER ONE SA',91084733000107,'RUA TIRADENTES JAD AGUA VERDE NUMERO 399',999631258,41,'POWER@GMAIL.COM',2);
INSERT INTO fornecedor (nome,cnpj,endereço,telefone,ddd_telefone,contato,cidade_id) VALUES ('RAIOVAC LTDA',33319727000195,'AVENIDA 7 DE SETEMBRO CENTRO NUMERO 659',34225539,43,'RAIOVAC@HOTMAIL.COM',3);
INSERT INTO fornecedor (nome,cnpj,endereço,telefone,ddd_telefone,contato,cidade_id) VALUES ('LIGHT POWER',17756318000108,'RUA PEDRO TAQUES CENTRO NUMERO 389',988116974,19,'LIGHT@HOTMAIL.COM',4);

SELECT id,nome,situaçao,cidade_id FROM fornecedor; /**Essa consulta permite avaliar os fornecedores que ainda estão na nossa lista de compras
A importancia da consulta é que podemos avaliar, se devemos pesquisar novos fornecedores, devido a possiveis faltas de opções de fornecedores**/

SELECT fornecedor.id,fornecedor.nome,fornecedor.endereço,cidade.nome,cidade.id,estado.sigla,estado.id
FROM fornecedor 
INNER JOIN cidade ON fornecedor.cidade_id = cidade.id
INNER JOIN estado ON cidade.estado_id = estado.id; /**Essa consulta permite avaliar os fornecedores, que estão mais proximos da empresa. 
A importancia dessa consulta é que podemos economizar com valores de fretes, consultando o produto que precisamos, que esteja mais proximo da empresa**/

CREATE TABLE compra(
id INT NOT NULL AUTO_INCREMENT
,desconto DECIMAL (9,2)
,sem_desconto DECIMAL (9,2)
,diferença_valor DECIMAL (9,2) AS (sem_desconto - desconto)/**Auxilia o financeiro da empresa a avaliar se o desconto é satisfatorio, em uma grande quantidade de compra de produtos**/
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,fornecedor_id INT NOT NULL
,CONSTRAINT pk_compra PRIMARY KEY (id)
,CONSTRAINT fk_compra_fornecedor FOREIGN KEY (fornecedor_id) REFERENCES fornecedor (id)
);

INSERT INTO compra (desconto,sem_desconto,fornecedor_id) VALUES (35.00,45.00,3);
INSERT INTO compra (desconto,sem_desconto,fornecedor_id) VALUES (100.00,120.00,1);
INSERT INTO compra (desconto,sem_desconto,fornecedor_id) VALUES (50.00,70.00,2);
INSERT INTO compra (desconto,sem_desconto,fornecedor_id) VALUES (15.00,30.00,4);
INSERT INTO compra (desconto,sem_desconto,fornecedor_id) VALUES (10.00,23.00,1);

SELECT id,desconto,data_cadastro,sem_desconto,fornecedor_id FROM compra; /**Consulta para saber qual o valor total das compras realizadas dentro do periodo.
A Importancia é a medição do fluxo de caixa**/

SELECT 
SUM(desconto) AS 'COMPRA COM DESCONTO'
,SUM(sem_desconto) AS 'COMPRA SEM DESCONTO'
,sum(sem_desconto) - SUM(desconto) AS 'ECONOMIA'
FROM compra; /**Consulta para saber a diferença de valores com as compras.
A Importancia é a medição da economia com as compras, que foi realizada no periodo**/


CREATE TABLE iten_compra(
compra_id INT NOT NULL 
,produto_id INT NOT NULL
,nome VARCHAR (200) NOT NULL
,quantidade INT NOT NULL
,preço DECIMAL (9,2)
,PRIMARY KEY (compra_id,produto_id)
);

INSERT INTO iten_compra (nome,quantidade,preço,compra_id,produto_id) VALUES ('Disjuntor Monopolar  25A',4,35.00,1,1);
INSERT INTO iten_compra (nome,quantidade,preço,compra_id,produto_id) VALUES ('Cabo de cobre flexível isolado',2,100.00,2,2);
INSERT INTO iten_compra (nome,quantidade,preço,compra_id,produto_id) VALUES ('Interruptor simples (1 módulo)',2,50.00,3,3);
INSERT INTO iten_compra (nome,quantidade,preço,compra_id,produto_id) VALUES ('Caixa retangular 4" x 2"',6,15.00,4,4);
INSERT INTO iten_compra (nome,quantidade,preço,compra_id,produto_id) VALUES ('Eletroduto de pvc',7,10.00,5,5);

DROP FUNCTION caculo_itenCompra;
DELIMITER //
CREATE FUNCTION caculo_itenCompra(quantidade INT, id INT) 
RETURNS DECIMAL (9,2) DETERMINISTIC
DETERMINISTIC
BEGIN
    DECLARE preço DECIMAL (9,2);
    DECLARE quantidade INT;
    DECLARE resultado DECIMAL (9,2);
    
	SELECT quantidade * preço INTO resultado FROM iten_compra WHERE quantidade = quantidade.iten_compra AND id = id.iten_compra;
	
    RETURN resultado;
END;
//
DELIMITER ;

SELECT caculo_itenCompra(1,2);

CREATE TABLE pagamento(
id INT NOT NULL AUTO_INCREMENT
,situaçao CHAR (1) NOT NULL DEFAULT 'E'
,numero_parcela INT
,desconto DECIMAL (9,2)
,sem_desconto DECIMAL (9,2)
,juros DECIMAL (9,2)
,total_com_juros DECIMAL (9,2)
,compra_id INT NOT NULL
,iten_caixa_id INT NOT NULL
,CONSTRAINT pk_pagamento PRIMARY KEY (id)
,CONSTRAINT fk_pagamento_compra FOREIGN KEY (compra_id) REFERENCES compra (id)
,CONSTRAINT fk_pagamento_iten_caixa FOREIGN KEY (iten_caixa_id) REFERENCES iten_caixa (id)
,CONSTRAINT coluna_situaçao_pagamento CHECK (situaçao IN ('E','N')) -- E = Efetuado ou N = Não Efetuado
,CONSTRAINT compra_unico UNIQUE (compra_id,iten_caixa_id)
);

CREATE TABLE categoria(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,descrição VARCHAR (300) NOT NULL
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,CONSTRAINT pk_categoria PRIMARY KEY (id)
);

CREATE TABLE produto_categoria(
produto_id INT NOT NULL 
,categoria_id INT NOT NULL 
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,PRIMARY KEY (produto_id,categoria_id)
);

CREATE TABLE tipo(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,descrição VARCHAR (300) NOT NULL
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,CONSTRAINT pk_tipo PRIMARY KEY (id)
);

CREATE TABLE produto_tipo(
produto_id INT NOT NULL 
,tipo_id INT NOT NULL 
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,PRIMARY KEY (produto_id,tipo_id)
);

CREATE TABLE produto(
id INT NOT NULL AUTO_INCREMENT
,nome VARCHAR (200) NOT NULL
,descriçao VARCHAR (200) NOT NULL
,nome_descriçao VARCHAR (400) AS (CONCAT(nome,' - ', descriçao))/** Coluna virtual que evita as incosistências de atualizações das duas colunas **/
,preço DECIMAL (9,2) NOT NULL
,estoque INT NOT NULL
,situaçao CHAR (1) NOT NULL DEFAULT 'D'
,vencimento DATE NOT NULL
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,CONSTRAINT pk_produto PRIMARY KEY (id)
,CONSTRAINT coluna_situaçao_produto CHECK (situaçao IN ('D','E')) -- D = Disponivel e E = Esgotado
);

INSERT INTO produto (nome,descriçao,preço,estoque,vencimento) VALUES ('Disjuntor Monopolar  25A','Curva C Siemens e ou equivalente',35.00,3,'2021-10-22');
INSERT INTO produto (nome,descriçao,preço,estoque,vencimento) VALUES ('Cabo de cobre flexível isolado','35 mm² anti-chama 0,6/1,0 kv cor preto',100.00,2,'2021-10-22');
INSERT INTO produto (nome,descriçao,preço,estoque,vencimento) VALUES ('Interruptor simples (1 módulo)','10a/250v incluindo suporte e placa',50.00,2,'2022-11-11');
INSERT INTO produto (nome,descriçao,preço,estoque,vencimento) VALUES ('Caixa retangular 4" x 2"','média (1,30 m do piso) PVC',15.00,6,'2020-09-01');
INSERT INTO produto (nome,descriçao,preço,estoque,vencimento) VALUES ('Eletroduto de pvc','rigido roscavel (1 1/2")',10.00,7,'2023-09-11');

SELECT * FROM produto;

DROP FUNCTION vidaUtilProduto;**/
DELIMITER //
CREATE FUNCTION vidaUtilProduto(id INT)
RETURNS INT DETERMINISTIC
BEGIN
	DECLARE tempo INT;
    DECLARE id INT;
	
    SELECT vencimento, data_cadastro, DATEDIFF(vencimento, data_cadastro) INTO tempo FROM produto WHERE produto.id = id;
    
	RETURN tempo;
END;
//
DELIMITER ;

# Invocando a função
SELECT *,vidaUtilProduto(produto.id)
FROM produto;

SELECT nome,estoque FROM produto; /**Consulta de disponibilidade de produtos em estoque. Sua importancia é enorme para realizar novas compras, 
pois assim saberemos quais produtos estão com poucas unidades em estoque**/

/**DROP FUNCTION nivel_estoque;**/
DELIMITER //
CREATE FUNCTION nivel_estoque(estoque INT)/**Função que faz o detalhamento sobre a situação da quantidade do estoque. A importancia desse tipo de consulta é o indicador de alerta do estoque,
registrando a mensagem  do nivel de estoque em relação as unidades de cada produto**/
RETURNS VARCHAR (150) DETERMINISTIC
BEGIN
    DECLARE resultado VARCHAR (150);
	DECLARE produto_estoque INT (1);
    
    SELECT estoque INTO resultado FROM produto WHERE produto_estoque = estoque;
    
	IF estoque <= 2 THEN
		BEGIN
			SET resultado = 'resosição';
		END;
    ELSEIF estoque <= 3 THEN
		BEGIN
			SET resultado = 'estoque baixo';
		END;
	ELSEIF produto_estoque <= 5 THEN
		BEGIN
			SET resultado = 'estoque médio';
		END;
	ElSE
		BEGIN
			SET resultado = 'estoque alto';
	END;
    END IF;
	RETURN resultado;
END;
//
DELIMITER ;

# Invocando a função
SELECT nome, nivel_estoque(produto.estoque)
FROM  produto;

SELECT *
FROM produto 
WHERE nome LIKE '%Disjuntor%'; /**Consulta de filtro em estoque. A importancia é a agilidade da procura do produto, 
para saber sobre a disponibilidade do produto**/

CREATE TABLE venda(
id INT NOT NULL AUTO_INCREMENT
,desconto DECIMAL (9,2)
,sem_desconto DECIMAL (9,2)
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,cliente_id INT NOT NULL
,funcionario_id INT NOT NULL
,CONSTRAINT pk_venda PRIMARY KEY (id)
,CONSTRAINT fk_venda_cliente FOREIGN KEY (cliente_id) REFERENCES cliente (id)
,CONSTRAINT fk_venda_funcionario FOREIGN KEY (funcionario_id) REFERENCES funcionario (id)
);

INSERT INTO venda (desconto, sem_desconto, data_cadastro,cliente_id,funcionario_id) VALUES (200.00,400.00,'2010-05-30',1,2);
INSERT INTO venda (desconto, sem_desconto, data_cadastro,cliente_id,funcionario_id) VALUES (350.00,450.00,'2010-06-05',2,3);
INSERT INTO venda (desconto, sem_desconto, data_cadastro,cliente_id,funcionario_id) VALUES (500.00,600.00,'2010-07-28',3,4);
INSERT INTO venda (desconto, sem_desconto, data_cadastro,cliente_id,funcionario_id) VALUES (1000.00,1075.00,'2010-08-29',4,1);


CREATE TABLE iten_venda(
venda_id INT NOT NULL 
,produto_id INT NOT NULL
,quantidade INT NOT NULL
,preco DECIMAL (9,2)
,desconto INT DEFAULT 0
,total DECIMAL(9,2) AS (quantidade * preco - desconto) STORED
,CONSTRAINT pk_iten_venda PRIMARY KEY (venda_id,produto_id)
,CONSTRAINT fk_iten_venda_venda FOREIGN KEY (venda_id) REFERENCES venda (id)
,CONSTRAINT fk_iten_venda_produto_id FOREIGN KEY (produto_id) REFERENCES produto (id)
);

INSERT INTO iten_venda (venda_id, produto_id,quantidade,preco,desconto) VALUES (1,1,1,400.00,200.00);
INSERT INTO iten_venda (venda_id, produto_id,quantidade,preco,desconto) VALUES (4,1,2,450.00,350.00);
INSERT INTO iten_venda (venda_id, produto_id,quantidade,preco,desconto) VALUES (2,1,5,600.00,500.00);
INSERT INTO iten_venda (venda_id, produto_id,quantidade,preco,desconto) VALUES (3,1,3,1075.00,1000.00);

/**DROP FUNCTION IF EXISTS verificar_desconto_venda;**/
DELIMITER //
CREATE FUNCTION verificar_desconto_venda(codigo_venda INT)
RETURNS BOOL DETERMINISTIC
BEGIN
	DECLARE valido BOOL DEFAULT TRUE;
    DECLARE limite_desconto DECIMAL(9,2);
	DECLARE preco DECIMAL(9,2);
	DECLARE quantidade INT;
    DECLARE desconto_geral DECIMAL(9,2);
	DECLARE desconto_unitario DECIMAL(9,2);
    DECLARE acabou BOOL DEFAULT FALSE;
    DECLARE cursor_iv CURSOR FOR
		SELECT
			iv.preco,
            iv.quantidade,
            iv.desconto
		FROM iten_venda iv
        WHERE iv.venda_id = codigo_venda;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET acabou = TRUE;    
        
	OPEN cursor_iv;
    loop_iv : LOOP
		FETCH cursor_iv INTO preco, 
        quantidade, desconto_geral;
        SET desconto_unitario = quantidade/desconto_geral;
        SET limite_desconto = preco * 0.5;
        IF desconto_unitario < limite_desconto THEN
			BEGIN
				SET valido = FALSE;
            END;
		END IF;
		IF acabou OR NOT valido THEN
			BEGIN
				LEAVE loop_iv;
			END;
		END IF;
	END LOOP;
    
    CLOSE cursor_iv;
    
	RETURN valido;
END ;
//
DELIMITER ;

SELECT verificar_desconto_venda(6);


CREATE TABLE recebimento(
id INT NOT NULL AUTO_INCREMENT
,situaçao CHAR (1) NOT NULL DEFAULT 'E'
,numero_parcela INT
,desconto DECIMAL (9,2)
,sem_desconto DECIMAL (9,2)
,juros DECIMAL (9,2)
,total_com_juros DECIMAL (9,2)
,data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
,venda_id INT NOT NULL
,iten_caixa_id INT NOT NULL
,CONSTRAINT pk_pagamento PRIMARY KEY (id)
,CONSTRAINT fk_recebimento_venda FOREIGN KEY (venda_id) REFERENCES venda (id)
,CONSTRAINT fk_recebimento_iten_caixa FOREIGN KEY (iten_caixa_id) REFERENCES iten_caixa (id)
,CONSTRAINT coluna_situaçao_recebimento CHECK (situaçao IN ('E','N')) -- E = Efetuado ou N = Não Efetuado
,CONSTRAINT compra_unico UNIQUE (venda_id,iten_caixa_id)
);





