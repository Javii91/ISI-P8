Plataphorma
===========

Meteor pet project created to teach my students the following Meteor functionality: 

* Collection's publish/subscribe 
* Deps.autorun 
* Meteor.methods/call 
* Integration of non-Meteor code in compatibility folder (HTML5 games Alien Invasion and Froot Wars)
* Usage of allow to control client access to collections

![ScreenShot](/screenshot.png)


Plataphorma offers the possibility to run 2 different HTML5 games: Alien Invasion and Froot Wars. 

On the right side of the screen the best players of each game are shown, updated in real time each time a signed in player finishes a game. If no game is selected, the best players overall are shown.

On the left side of the screen a chatroom for the current game is available. Only signed in users can post messages. If no game is selected a general chatroom is shown.

The original code of the two HTML5 games integrated in this project is available here:
* Alien Invasion: https://github.com/cykod/AlienInvasion
* Froot Wars: http://www.wrox.com/WileyCDA/WroxTitle/Professional-HTML5-Mobile-Game-Development.productCd-1118301323,descCd-DOWNLOAD.html

Bootstrap style (file bootstrap.min.css) provided by http://bootswatch.com


Running the project
-------------------

A live version of this code is running here: http://plataphorma.meteor.com

To run the project locally, clone the repo and run ```meteor``` inside it. You can see in .meteor/packages that this Meteor project uses these packages:
* ```meteor remove autopublish```
* ```meteor remove insecure```
* ```meteor add bootstrap```
* ```meteor add accounts-ui```
* ```meteor add accounts-password```


1) Click en el boton de juego

--> index.html
<div id="games" class="btn-group">

    <input id="none" class="btn" type="button" value="None"></input>
    <input id="FrootWars" class="btn btn-primary" type="button" value="FrootWars"></input>
    <input id="AlienInvasion" class="btn btn-primary" type="button" value="AlienInvasion"></input>

</div>

Los click en las pestañas llama a:

--> client.js
Template.choose_game.events = {
    'click #AlienInvasion': function () {
	$('#gamecontainer').hide();
	$('#container').show();
	var game = Games.findOne({name:"AlienInvasion"});
	Session.set("current_game", game._id);
    },
    'click #FrootWars': function () {
	$('#container').hide();
	$('#gamecontainer').show();
	var game = Games.findOne({name:"FrootWars"});
	Session.set("current_game", game._id);
    },
    'click #none': function () {
	$('#container').hide();
	$('#gamecontainer').hide();
	Session.set("current_game", "none");
    }
}

Esconde las pestañas de los botones que no se han pulsado y muestra la del boton pulsado y guardas en current_game el juego actual


2) Se escribe mensaje chat sin estar autenticado

--> client.js
Template.input.events = {
    'keydown input#message' : function (event) {
	if (event.which == 13) { 
	    if (Meteor.userId()){
		...
	    }
	    else {
		$("#login-error").show();
	    }
	}
    }
}

Se muestra un error de login error que dice You must be signed in to post messages.

3) Se escribe mensaje chat estando autenticado


--> client.js
Template.input.events = {
    'keydown input#message' : function (event) {
	if (event.which == 13) { 
	    if (Meteor.userId()){
		var user_id = Meteor.user()._id;	    
		var message = $('#message');
		if (message.value != '') {
		    Messages.insert({
			user_id: user_id,
			message: message.val(),
			time: Date.now(),
			game_id: Session.get("current_game")
		    });
		    message.val('')
		}
	    }
	    else {
		$("#login-error").show();
	    }
	}
    }
}

Cuando estas logueado, coge el id del jugador y el mensaje que ha escrito, si el mensaje no esta vacio lo inserta en la base de datos con los campos id_usuario, mensaje del usuario, fecha del mensaje y chat del juego al que se ha escrito, por ultimo vacia el input donde se escribe



4) Se termina partida con puntacion sin estar autenticado


--> game.js de AlienInvasion
var winGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You win!", 
                                    "Press fire to play again",
                                    playGame));
};


// Llamada cuando la nave del jugador ha sido alcanzada, para
// finalizar el juego
var loseGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You lose!", 
                                    "Press fire to play again",
                                    playGame));
};

--> game.js de FrootWars
showEndingScreen:function(){
			game.stopBackgroundMusic();				
			if (game.mode=="level-success"){			
				if(game.currentLevel.number<levels.data.length-1){
					$('#endingmessage').html('Level Complete. Well Done!!!');
					$("#playnextlevel").show();
				} else {
					$('#endingmessage').html('All Levels Complete. Well Done!!!');
					$("#playnextlevel").hide();

   				        Meteor.call("matchFinish", Session.get("current_game"), game.score);				        

				}
			} else if (game.mode=="level-failure"){			
   			    Meteor.call("matchFinish", Session.get("current_game"), game.score);				        
			    
			    $('#endingmessage').html('Failed. Play Again?');
			    $("#playnextlevel").hide();
			}		
	
			$('#endingscreen').show();


Cuando se acaba una partida se llama al metodo matchFinish.

--> Server.js
Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId)
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
});

Se declara un metodo para ser llamado por el usuario y guardar la puntuacion en la base de datos, no está autenticado, no entra en el if, por lo tanto no guarda la puntuacion


5) Se termina partida con puntacion estando autenticado

--> game.js de AlienInvasion
var winGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You win!", 
                                    "Press fire to play again",
                                    playGame));
};


// Llamada cuando la nave del jugador ha sido alcanzada, para
// finalizar el juego
var loseGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You lose!", 
                                    "Press fire to play again",
                                    playGame));
};

--> game.js de FrootWars
showEndingScreen:function(){
			game.stopBackgroundMusic();				
			if (game.mode=="level-success"){			
				if(game.currentLevel.number<levels.data.length-1){
					$('#endingmessage').html('Level Complete. Well Done!!!');
					$("#playnextlevel").show();
				} else {
					$('#endingmessage').html('All Levels Complete. Well Done!!!');
					$("#playnextlevel").hide();

   				        Meteor.call("matchFinish", Session.get("current_game"), game.score);				        

				}
			} else if (game.mode=="level-failure"){			
   			    Meteor.call("matchFinish", Session.get("current_game"), game.score);				        
			    
			    $('#endingmessage').html('Failed. Play Again?');
			    $("#playnextlevel").hide();
			}		
	
			$('#endingscreen').show();


Cuando se acaba una partida se llama al metodo matchFinish.

--> Server.js
Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId)
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
});

Se declara un metodo para ser llamado por el usuario y guardar la puntuacion en la base de datos, se guarda en la base de datos Matches el id del usuario, la fecha en que ha conseguido la puntuacion, los puntos conseguidos y el juego en que se ha conseguido.

6) ¿Que sale en consola cuando te autenticas (sign in)?

[12:24:19.581] "current user: null"
[12:24:19.581] "current user: TH5iSzcypxbfv9LJD"

--> Client.js
var currentUser = null;
Deps.autorun(function(){
    console.log("current user: " + currentUser);
    currentUser = Meteor.userId();
    console.log("current user: " + currentUser);
});

Te sale el valor por defecto de no estar autenticado, y a continuacion saca el id de usuario autenticado y te lo muestra.



7) ¿Que sale en consola cuando sale (Sign out)?


[12:27:19.820] "current user: TH5iSzcypxbfv9LJD"
[12:27:19.820] "current user: null"

--> Client.js

Deps.autorun(function(){
    console.log("current user: " + currentUser);
    currentUser = Meteor.userId();
    console.log("current user: " + currentUser);
});

Aparece la ultimo id del usuario que ha quedado guardado, cuando vuelve a comprobar el id del usuario, ya no estas autenticado por lo que se pone a null, y luego escribe null





