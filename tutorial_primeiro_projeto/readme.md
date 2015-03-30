<!-- baseado em: https://docs.angularjs.org/tutorial -->

# Tutorial Leilão boxJS

Uma das formas mais comum de nós programadores aprendermos a programar com uma nova linguagem ou um novo framework, é, com certeza, usando-o para desenvolver
alguma aplicação, então, para fixar o que foi ensinado sobre o boxJS até o momento, propomos a construção de uma simples aplicação Web de leilões online.


## Sumário


## Antes de começar

### Bootstrap e AngularJS

É interessante que, antes de começarmos, você tenha tudo instalado e esteja familiarizado com as outras ferramentas que utilizaremos, são elas:

* [Bootstrap](http://getbootstrap.com/)
* [AngularJS](https://angularjs.org/)

Apesar dessas ferramentas serem amplamente usadas neste tutorial para desenvolver o front-end, acreditamos que seja póssivel seguí-lo apenas com um 
conhecimento básico delas.


### Front End

Como nosso objetivo é focar no aprendizado do boxJS, a parte front-end do nosso leilão já está pronta para download [clicando aqui](https://www.dropbox.com/s/xyo3b8cgra6oq4u/leilaoBoxJS.rar?dl=0), 
mas você esta livre caso queira implementá-la, apenas certifique-se que o front-end esteja utilizando corretamente o boxJS (para isso recomendamos que você faça o 
download e veja como nós acessamos o back-end).


## Back End com boxJS

Nosso tutorial começa, de fato, a partir de agora, mas consideramos que você seguiu e leu todos os passos até aqui e que já está com seu [front end pronto e o boxJS funcionando, com alguns setups
básicos](#front-end), conforme ensinado nos tutoriais anteriores.

### `routes.js`

O nosso primeiro passo é arrumar as rotas. Conforme podemos ver no nosso serviço de acesso ao banco abaixo, nossa aplicação espera acessar as urls `boxjs/leilao/inserir`, `boxjs/leilao/buscar`, 
`boxjs/leilao/darlance`, `boxjs/leilao/fechar`, `boxjs/usuario/tela`, `boxjs/usuario/buscar`, `boxjs/usuario/inserir`.



~~~ javascript

leilaoApp.factory("databaseSvc", ['$http','$q', function ($http,$q) {
	return {
		leilao: {
			insereLeilao: function (doc) {
			    return $http.post("boxjs/leilao/inserir", {docToInsert: doc});
			},
			buscaLeilao: function (busca, campos) {
				return $http.post("boxjs/leilao/buscar", { query: busca, fields: campos});
			},
			darLance: function (identificador, lanceAdicionado) {
				return $http.post("boxjs/leilao/darlance", { id: identificador, lance: lanceAdicionado});
			},
			fecharLeilao: function (identificador) {
				return $http.post("boxjs/leilao/fechar", { id: identificador });
			}
		},
		usuario: {
			tela: function() {
				return $http.post("boxjs/usuario/tela", {});
			},
			buscaUsuario: function (busca, campos) {
				return $http.post("boxjs/usuario/buscar", { query: busca, fields: campos});
			},
			insereUsuario: function (doc) {
			    return $http.post("boxjs/usuario/inserir", {docToInsert: doc});
			}
		}
	};
}]);

~~~


Além disso, o serviço de autenticação, conforme o código abaixo, espera acessar a url `boxjs/svr/login`.


~~~ javascript

leilaoApp.factory("authenticationSvc", ['$http','$q','$window','userInfoFac',function($http, $q, $window,userInfoFac) {

	function setCookie(key, value, tempoParaExpirarMinutos) {
        var mins = tempoParaExpirarMinutos;
        if (isNaN(mins)) {
            mins = 60;
        }
        var now = new Date();
        var exp = new Date(now.getTime() + mins*mins*1000);
        var status = '';
        $window.document.cookie = key + '=' + value + '; expires='+exp.toUTCString();
    }
    
    function deleteCookie(key) {
        $window.document.cookie = key + '=' + "; expires=Thu, 01 Jan 1970 00:00:00 UTC";
    }
 
    function login(userName, password) {
		deleteCookie("userInfo");
	    var deferred = $q.defer();
	    $http.post("boxjs/svr/login", {
	        user: userName,
	        pass: $.md5(password)
	    }).then(function(result) {
	    	if(result.status != 200) {
	    		deferred.reject(result.data.reason);
	    	}
	        userInfo = {
	            accessToken: result.data.accessToken,
	            userName: result.data.userName,
	            timestamp: result.data.timestamp
	        };
	        userInfoFac.setUserInfo(userInfo);
	        setCookie("userInfo", JSON.stringify(userInfo));
	        deferred.resolve(userInfo);
	    }, function(error) {
	        deferred.reject(error);
	    });
	    return deferred.promise;
    }
	
	return {
		setCookie: setCookie,
		deleteCookie: deleteCookie,
		login: login		
	};
}]);

~~~


Uma vez que já sabemos as URLs que serão acessadas, podemos criar as rotas, já que não é considerada uma boa prática deixar os arquivos no diretório inicial do boxJS. Usaremos, então, a 
pasta `actions` para colocar o que será executado pelo boxJS, o primeiro passo é criar os três arquivos, `usuario.js`, `leilao.js`, `svr.js` (sim, esses arquivos permanecem vazios, por enquanto,
pois os implanteremos mais tarde!), dentro desta pasta. Com isso feito podemos, finalmente, configurar nossas rotas, é importante lembrar que isso é feito dentro do arquivo `routes.js` que se 
encontra dentro da pasta `modules` e que a adição de uma rota segue o padrão `Router.add( 'nomeDaURL', 'caminho/para/o/arquivo')`, deixando nosso arquivo `routes.js` da seguinte forma:

~~~ javascript

Router.add( 'usuario', 'actions/usuario');

Router.add( 'leilao', 'actions/leilao');

Router.add( 'svr', 'actions/svr');

~~~


### `leilao.js` e `usuario.js`

O primeiro passo parar implementar estes dois arquivos é saber quais serão os seus métodos, conforme vimos na criação do `routes.js`, o front end irá acessar as URLs: `boxjs/leilao/inserir`, `boxjs/leilao/buscar`, 
`boxjs/leilao/darlance`, `boxjs/leilao/fechar`, `boxjs/usuario/tela`, `boxjs/usuario/buscar`, `boxjs/usuario/inserir`. Logo, nossos métodos são:

* leilao
  * [inserir](#leilaoinserir)
  * [buscar](#leilaobuscar)
  * [darlance](#leilaodarlance)
  * [fechar](#leilaofechar)
* usuario
  * [tela](#usuariotela)
  * [buscar](#usuariobuscar)
  * [inserir](#usuarioinserir)



#### leilao/inserir

Objetivo: Inserir um novo leilão no banco.

Parametros: Leilão a ser inserido.

Retorna: Informa se a inserção falhou ou não.

Primitivas:

- Status do novo leilão deve ser 'aberto'

- Não deve existir lances.

- O ID deve ser incremental.

Código:

~~~ javascript
inserir: function (params,request,response) {
	var colLeilao = db.leilaobox().getCollection("leiloes");
	var newLeil = params.docToInsert;
	newLeil.status = "aberto";
	newLeil.lances = [];
	newLeil._id = db.leilaobox().eval("getNextSequence('leilaoid')");
	var insrt = colLeilao.insert(newLeil);
	print("-- inserted leilao.js [finished] --------------------------------------------");
	response.setStatus(200);
	response.write(JSON.stringify({error: false, data: insrt}));
}
~~~

OBS: Para que o ID seja incremental adicionamos a função getNextSequence(name) ao banco que usa o comando findAndModify para incrementar o id de forma atômica.

~~~ javascript
function getNextSequence(name) {
   var ret = db.counters.findAndModify(
          {
            "query": { "_id": name },
            "update": { $inc: { "seq": 1 } },
            "new": true
          }
   );

   return ret.seq;
}
~~~



#### leilao/buscar

Objetivo: Buscar leilôes do banco.

Parametros: Query a ser executada e campos a serem retornado.

Retorna: Array de leilões resultantes da execução da query com os campos especificados.

Código:

~~~ javascript
buscar: function (params,request,response) {
	var colLeilao = db.leilaobox().getCollection("leiloes");
	var search = colLeilao.find(params.query,params.fields);
	print("-- search leilao.js [finished] --------------------------------------------");
	response.setStatus(200);
	response.write(JSON.stringify({error: false, data: search.toArray()}));
}
~~~


#### leilao/darlance

Objetivo: Adicionar um lance a um leilão do banco.

Parametros: ID do leilão que receberá o lance e o lance.

Retorna: Informa se a adição do lance falhou ou não.

Primitivas:

- Para um lance ser aceito ele deve ser maior que o mínimo e maior que o lance anterior, caso ele exista.

Código:

~~~ javascript
darlance: function (params,request,response) {
	var colLeilao = db.leilaobox().getCollection("leiloes");
	var search = colLeilao.find({ "_id": params.id });
	var leilaoToLance = search.next();
	if( params.lance.valor < leilaoToLance.lanceInicial || ( leilaoToLance.ultimoLance && leilaoToLance.ultimoLance > params.lance.valor ) ) {
		response.setStatus(400);
		print("-- failed dar lance leilao.js [finished] --------------------------------------------");
		response.write(JSON.stringify({error: true, data: "Lance abaixo do minimo."}));
		return;
}
	var upd = colLeilao.update( { "_id": params.id } , { "$push": { "lances": params.lance }, "$set": { "ultimoLance": params.lance.valor } });
	print("-- dar lance leilao.js [finished] --------------------------------------------");
	response.setStatus(200);
	response.write(JSON.stringify({error: false, data: upd}));
},
~~~



#### leilao/fechar

Objetivo: Fechar um leilão do banco.

Parametros: ID do leilão a ser fechado.

Retorna: Leilão fechado, caso o fechamento ocorra.

Primitivas:

- Testar se existe um leilão com o ID indicado.

- Colocar no leilão os dados do lance vencedor.

- Mudar o status do leilão para 'fechado'.


Código:

~~~ javascript
fechar: function (params,request,response) {
	var colLeilao = db.leilaobox().getCollection("leiloes");
	var search = colLeilao.find({ "_id": params.id });
	if(!search.hasNext()) {
		response.setStatus(400);
		print("-- failed close leilao.js [finished] --------------------------------------------");
		response.write(JSON.stringify({error: true, data: "Nao ha leiloes com este ID"}));
		return;
	}
	var leilaoFechar = search.next();
	var lanceGanhador = leilaoFechar.lances[leilaoFechar.lances.length-1];
	leilaoFechar.vencedor = lanceGanhador.usuario;
	leilaoFechar.status = "fechado";
	var upd = colLeilao.update( { "_id": params.id } , leilaoFechar );
	print("-- closed leilao.js [finished] --------------------------------------------");
	response.setStatus(200);
	response.write(JSON.stringify({error: false, data: leilaoFechar}));
}
~~~




#### usuario/tela

Objetivo: Verificar se a tela de usuario pode ser acessada pelo usuario atual.

Retorna: Libera o acesso para a tela.

Código:

~~~ javascript
tela: function (params,request,response) {
	print("-- tela usuario.js [finished] --------------------------------------------");
	response.setStatus(200);
	response.write(JSON.stringify({error: false, data: "OK"}));
}
~~~

OBS: Este método fará mais sentido quando o `security.js` for implementado.




#### usuario/inserir

Objetivo: Inserir um novo usuário ao banco.

Parametros: Usuário a ser inserido.

Retorna: Informa se a inserção do usuario falhou ou não.

Primitivas:

- Setar o tipo do usuário para 'basic'.


Código:

~~~ javascript
inserir: function (params,request,response) {
	var colUsuario = db.leilaobox().getCollection("usuarios");
	var newUsr = params.docToInsert;
	newUsr.tipo = "basic";
	try {
		var insrt = colUsuario.insert(newUsr);
	} catch (e) {
		response.setStatus(400);
		print("-- failed insert usuario.js [finished] --------------------------------------------");
		response.write(JSON.stringify({error: true, data: e.toString()}));
		return;
	}
	print("-- inserted usuario.js [finished] --------------------------------------------");
	response.setStatus(200);
	response.write(JSON.stringify({error: false, data: insrt}));
}
~~~


#### usuario/buscar

Objetivo: Buscar usuários no banco.

Parametros: Query a ser executada e campos a serem retornado.

Retorna: Array de usuários resultantes da execução da query com os campos especificados.

Código:

~~~ javascript
buscar: function (params,request,response) {
	var colUsuario = db.leilaobox().getCollection("usuarios");
	var search = colUsuario.find(params.query,params.fields);
	print("-- search usuario.js [finished] --------------------------------------------");
	response.setStatus(200);
	var resu = search.toArray();
	response.write(JSON.stringify({error: false, data: resu}));
}
~~~




Agora basta que juntemos todos esses métodos em seus respectivos arquivos, utilizando um objeto exports


~~~ javascript

//leilao.js

var colLeilao = db.leilaobox().getCollection("leiloes");

exports = {
	darlance: function (params,request,response) {
		var search = colLeilao.find({ "_id": params.id });
		var leilaoToLance = search.next();
		if( params.lance.valor < leilaoToLance.lanceInicial || ( leilaoToLance.ultimoLance && leilaoToLance.ultimoLance > params.lance.valor ) ) {
			response.setStatus(400);
			print("-- failed dar lance leilao.js [finished] --------------------------------------------");
			response.write(JSON.stringify({error: true, data: "Lance abaixo do minimo."}));
			return;
    	}
		var upd = colLeilao.update( { "_id": params.id } , { "$push": { "lances": params.lance }, "$set": { "ultimoLance": params.lance.valor } });
		print("-- dar lance leilao.js [finished] --------------------------------------------");
		response.setStatus(200);
		response.write(JSON.stringify({error: false, data: upd}));
	},
	buscar: function (params,request,response) {
		var search = colLeilao.find(params.query,params.fields);
		print("-- search leilao.js [finished] --------------------------------------------");
		response.setStatus(200);
		response.write(JSON.stringify({error: false, data: search.toArray()}));
	},
	inserir: function (params,request,response) {
		var newLeil = params.docToInsert;
		newLeil.status = "aberto";
		newLeil.lances = [];
		newLeil._id = db.leilaobox().eval("getNextSequence('leilaoid')");
		var insrt = colLeilao.insert(newLeil);
		print("-- inserted leilao.js [finished] --------------------------------------------");
		response.setStatus(200);
		response.write(JSON.stringify({error: false, data: insrt}));
	},
	fechar: function (params,request,response) {
		var search = colLeilao.find({ "_id": params.id });
		if(!search.hasNext()) {
			response.setStatus(400);
			print("-- failed close leilao.js [finished] --------------------------------------------");
			response.write(JSON.stringify({error: true, data: "Nao ha leiloes com este ID"}));
			return;
		}
		var leilaoFechar = search.next();
		var lanceGanhador = leilaoFechar.lances[leilaoFechar.lances.length-1];
		leilaoFechar.vencedor = lanceGanhador.usuario;
		leilaoFechar.status = "fechado";
		var upd = colLeilao.update( { "_id": params.id } , leilaoFechar );
		print("-- closed leilao.js [finished] --------------------------------------------");
		response.setStatus(200);
		response.write(JSON.stringify({error: false, data: leilaoFechar}));
	}
}

~~~


~~~ javascript

//usuario.js

var colUsuario = db.leilaobox().getCollection("usuarios");

exports = {
	tela: function (params,request,response) {
		print("-- tela usuario.js [finished] --------------------------------------------");
		response.setStatus(200);
		response.write(JSON.stringify({error: false, data: "OK"}));
	},
	buscar: function (params,request,response) {
		var search = colUsuario.find(params.query,params.fields);
		print("-- search usuario.js [finished] --------------------------------------------");
		response.setStatus(200);
		var resu = search.toArray();
		response.write(JSON.stringify({error: false, data: resu}));
	},
	inserir: function (params,request,response) {
		var newUsr = params.docToInsert;
		newUsr.tipo = "basic";
		try {
			var insrt = colUsuario.insert(newUsr);
		} catch (e) {
			response.setStatus(400);
			print("-- failed insert usuario.js [finished] --------------------------------------------");
			response.write(JSON.stringify({error: true, data: e.toString()}));
			return;
		}
		print("-- inserted usuario.js [finished] --------------------------------------------");
		response.setStatus(200);
		response.write(JSON.stringify({error: false, data: insrt}));
	}
}

~~~

Há uma pequena diferença entre os códigos de cada método e o código que os unifica. Como o acesso a collection a qual o arquivo é referente será feito por praticamente todos os seus métodos
colocamos esse acesso fora de um método especifico.

// ensinar a programar o svr.js --> adicionando o modulo aes.js

// ensinar a programar o security.js


// profit


