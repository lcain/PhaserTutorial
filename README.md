### Phaser Tutorial

This tutorial was written by Alvin Ourrad and Richard Davey. I have created this modified version to teach the Web Development Immersive 12 class at General Assembly Sydney.

Please see the following for the original:
http://phaser.io/tutorials/making-your-first-phaser-game

I have left in the original files for the Phaser tutorial and added my own for demonstration purposes.

We'll be working in the wdiTutorial.js for this demonstration. This file is linked to the index.html file, which is blank and will not be modified any further.

For Phaser to work it needs to be run on a server. Let's start our server with python. Once you are in the tutorial directory in your termial type:
```
$ python -m SimpleHTTPServer
```
Open a new browser window and go to localhost:8000

Open your wdiTutorial.js file and follow the steps below.

####Step one:
Copy the following code and add it to wdiTutorial.js:
```
var game = new Phaser.Game(800, 600, Phaser.AUTO, '', { preload: preload, create: create, update: update });
```

####Step two:
Add the preload/create/update functions below:

```
function preload() {

    game.load.image('sky', 'assets/sky.png');
    game.load.image('ground', 'assets/platform.png');
    game.load.image('star', 'assets/star.png');
    game.load.spritesheet('dude', 'assets/dude.png', 32, 48);

}

function create() {
}

function update() {
}
```

####Step three:
Add a star to your create function.
```
function create() {
  game.add.sprite(0, 0, 'star');
}
```

####Step four:
Go ahead and delete that star sprite. We'll add more later.
Under your preload function, lets start adding variables. 

```
var platforms;
```
Now inside of your create function, lets add some magic Phaser physics. We'll start talking about groups in Phaser here, and add some platforms as well as a background.

```
function create() {

    //  We're going to be using physics, so enable the Arcade Physics system
    game.physics.startSystem(Phaser.Physics.ARCADE);

    //  A simple background for our game
    game.add.sprite(0, 0, 'sky');

    //  The platforms group contains the ground and the 2 ledges we can jump on
    platforms = game.add.group();

    //  We will enable physics for any object that is created in this group
    platforms.enableBody = true;

    // Here we create the ground.
    var ground = platforms.create(0, game.world.height - 64, 'ground');

    //  Scale it to fit the width of the game (the original sprite is 400x32 in size)
    ground.scale.setTo(2, 2);

    //  This stops it from falling away when you jump on it
    ground.body.immovable = true;

    //  Now let's create two ledges
    var ledge = platforms.create(400, 400, 'ground');

    ledge.body.immovable = true;

    ledge = platforms.create(-150, 250, 'ground');

    ledge.body.immovable = true;
    
}
```

####Step Five:
Lets add a player sprite! 
We need to add another global variable for our player. Let's add one below our platforms variable.
```
var platforms;
var player;
```
Now back in our create function add the following code to create your player. We'll need to add physics and animations for the player as well.

```
 // The player and its settings
    player = game.add.sprite(32, game.world.height - 150, 'dude');

    //  We need to enable physics on the player
    game.physics.arcade.enable(player);

    //  Player physics properties. Give the little guy a slight bounce.
    player.body.bounce.y = 0.2;
    player.body.gravity.y = 300;
    player.body.collideWorldBounds = true;

    //  Our two animations, walking left and right.
    player.animations.add('left', [0, 1, 2, 3], 10, true);
    player.animations.add('right', [5, 6, 7, 8], 10, true);
```

####Step Six:
We need to tell Phaser that our Player can't walk through platforms.
```
function update() {
    
     //  Collide the player and the stars with the platforms
    game.physics.arcade.collide(player, platforms);
}
``` 

####Step Seven:
Let's create movement for our player! In our update function add the following:
```

    //  Reset the players velocity (movement)
    player.body.velocity.x = 0;

    if (cursors.left.isDown)
    {
        //  Move to the left
        player.body.velocity.x = -150;

        player.animations.play('left');
    }
    else if (cursors.right.isDown)
    {
        //  Move to the right
        player.body.velocity.x = 150;

        player.animations.play('right');
    }
    else
    {
        //  Stand still
        player.animations.stop();

        player.frame = 4;
    }
    
    //  Allow the player to jump if they are touching the ground.
    if (cursors.up.isDown && player.body.touching.down)
    {
        player.body.velocity.y = -350;
    }
```
Now we'll need one more thing to let our little dude move, we need to tell Phaser what a cursor is. Let's add another global variable for cursor. 
```
var player;
var platforms;
var cursors;
```
But what does cursors mean? Let's define this at the bottom of our create function with some more Phaser magic.
```
 //  Our controls.
    cursors = game.input.keyboard.createCursorKeys();
```
We can move!

####Step Eight:
Remember that star we made? Let's make more!
First let's make a global variable for them.
```
var player;
var platforms;
var cursors;
var stars;
```
Now let's add some stars in our create function.
```
//  Finally some stars to collect
    stars = game.add.group();

    //  We will enable physics for any star that is created in this group
    stars.enableBody = true;

    //  Here we'll create 12 of them evenly spaced apart
    for (var i = 0; i < 12; i++)
    {
        //  Create a star inside of the 'stars' group
        var star = stars.create(i * 70, 0, 'star');

        //  Let gravity do its thing
        star.body.gravity.y = 300;

        //  This just gives each star a slightly random bounce value
        star.body.bounce.y = 0.7 + Math.random() * 0.2;
    }
```
Oh no! What happened?! They fell through the earth!
Phaser doesn't know that the stars should collide with our platforms. Let's tell it to do that now. In our update function just below our collision for our player and our platform, let's add one for our stars and platforms as well. 
```
    game.physics.arcade.collide(stars, platforms);
```
Now we have happy bouncing stars!

####Step Nine
Our stars are pretty cool. But it would be nice if we could collect them too.
Below our collision handlers in our update function, let's add the following.
```
//  Checks to see if the player overlaps with any of the stars, if he does call the collectStar function
    game.physics.arcade.overlap(player, stars, collectStar, null, this);
```
Right now we are calling a function called 'collectStar', so we need to create the collectStar function for Phaser to read. Below our update function, let's create a new function.
```
function collectStar (player, star) {
    
    // Removes the star from the screen
    star.kill();

}
```
And now we can collect stars!

####Step 10:
But what if we want to keep track of how many stars we collected? Let's create a score board!
First we need to make a few more global variables. Let's make a score variable, and a scoreText variable.
```
var player;
var platforms;
var cursors;
var stars;
var score = 0;
var scoreText;
```
In our collectStar function, let's add this and tell Phaser that we want every star to be worth 10 points.
```
//  Add and update the score
    score += 10;
    scoreText.text = 'Score: ' + score;
```
But what about the scoreText?
We need to tell Phaser what the scoreText is and where it goes. So let's do that back in our create function by adding the following.
```
//  The score
    scoreText = game.add.text(16, 16, 'score: 0', { fontSize: '32px', fill: '#000' });
```

#Congratulations!
You've made your very first game!








Oh no, did something go wrong? Copy and paste the following into your wdiTutorial.js file for the full working version.

```
var game = new Phaser.Game(800, 600, Phaser.AUTO, '', { preload: preload, create: create, update: update });


function preload() {

    game.load.image('sky', 'assets/sky.png');
    game.load.image('ground', 'assets/platform.png');
    game.load.image('star', 'assets/star.png');
    game.load.spritesheet('dude', 'assets/dude.png', 32, 48);

}

var player;
var platforms;
var cursors;
var stars;
var score = 0;
var scoreText;


function create() {

    //  We're going to be using physics, so enable the Arcade Physics system
    game.physics.startSystem(Phaser.Physics.ARCADE);

    //  A simple background for our game
    game.add.sprite(0, 0, 'sky');

    //  The platforms group contains the ground and the 2 ledges we can jump on
    platforms = game.add.group();

    //  We will enable physics for any object that is created in this group
    platforms.enableBody = true;

    // Here we create the ground.
    var ground = platforms.create(0, game.world.height - 64, 'ground');

    //  Scale it to fit the width of the game (the original sprite is 400x32 in size)
    ground.scale.setTo(2, 2);

    //  This stops it from falling away when you jump on it
    ground.body.immovable = true;

    //  Now let's create two ledges
    var ledge = platforms.create(400, 400, 'ground');

    ledge.body.immovable = true;

    ledge = platforms.create(-150, 250, 'ground');

    ledge.body.immovable = true;

     // The player and its settings
    player = game.add.sprite(32, game.world.height - 150, 'dude');

    //  We need to enable physics on the player
    game.physics.arcade.enable(player);

    //  Player physics properties. Give the little guy a slight bounce.
    player.body.bounce.y = 0.2;
    player.body.gravity.y = 300;
    player.body.collideWorldBounds = true;

    //  Our two animations, walking left and right.
    player.animations.add('left', [0, 1, 2, 3], 10, true);
    player.animations.add('right', [5, 6, 7, 8], 10, true);

    //  Our controls.
    cursors = game.input.keyboard.createCursorKeys();

    //  Finally some stars to collect
    stars = game.add.group();

    //  We will enable physics for any star that is created in this group
    stars.enableBody = true;

    //  Here we'll create 12 of them evenly spaced apart
    for (var i = 0; i < 12; i++)
    {
        //  Create a star inside of the 'stars' group
        var star = stars.create(i * 70, 0, 'star');

        //  Let gravity do its thing
        star.body.gravity.y = 300;

        //  This just gives each star a slightly random bounce value
        star.body.bounce.y = 0.7 + Math.random() * 0.2;
    }

    //  The score
    scoreText = game.add.text(16, 16, 'score: 0', { fontSize: '32px', fill: '#000' });
}

function update() {

     //  Collide the player and the stars with the platforms
    game.physics.arcade.collide(player, platforms);
    game.physics.arcade.collide(stars, platforms);

    //  Checks to see if the player overlaps with any of the stars, if he does call the collectStar function
    game.physics.arcade.overlap(player, stars, collectStar, null, this);

    //  Reset the players velocity (movement)
    player.body.velocity.x = 0;

    if (cursors.left.isDown)
    {
        //  Move to the left
        player.body.velocity.x = -150;

        player.animations.play('left');
    }
    else if (cursors.right.isDown)
    {
        //  Move to the right
        player.body.velocity.x = 150;

        player.animations.play('right');
    }
    else
    {
        //  Stand still
        player.animations.stop();

        player.frame = 4;
    }
    
    //  Allow the player to jump if they are touching the ground.
    if (cursors.up.isDown && player.body.touching.down)
    {
        player.body.velocity.y = -350;
    }
}

function collectStar (player, star) {
    
    // Removes the star from the screen
    star.kill();

    
    //  Add and update the score
    score += 10;
    scoreText.text = 'Score: ' + score;

}
```