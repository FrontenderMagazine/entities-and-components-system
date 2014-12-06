# Введение в сущности и компоненты

Imagine you are creating a roleplaying game. You can choose between 4 playable
characters that differ greatly between themselves - it should be simple enough 
to implement it in a classical way (or even as simple objects). For example:

    var Hero = function() {
        this.health = 100;
        this.power = 20;
        this.sprite = 'hero.png';
        this.weapon = 'spear';
        this.damage = 20;
    };
    
    var hero1 = newHero();

## No Country for Old Orcs

Cool, that works. It doesn’t feel ideal though. What if we add new characters
later? And what about the enemies?

Let’s forget the extra character for now (leave it to a DLC) and let’s
focus on the enemies. Suppose we want our orcs to be generated at random (yay 
diversity) from a set of attributes - maybe we want some that are strong and 
some that are weak, maybe some with bows, others with axes.

An awesome way to approach this problem is by using an entities and components
system.

![Avengers, assemble!][1]

## Yo Dawg I heard you like components

Think of components as the smallest building blocks that, assembled together,
form an entity. For our orc-folk, maybe we could do this:

    var components = {
      orc: function(entity){
        entity.health = 100;
        entity.humanoid = true;
        entity.color =  'green';
      },
    
      strong: function(entity){
        if(entity.health !== null)
          entity.health = entity.health*1.25;
      },
      
      axe: function(entity){
        entity.weapon = 'Axe';
        entity.damage = 5;
        entity.critMulti = 4;
        entity.critChance = 20;
      }
    };

Then if we do something like this:

    var orc1 = {};
    components.orc(orc1);
    components.strong(orc1);
    components.axe(orc1);

We will have an armed, tough orc ready! How cool and simple is this? If we
wanted to create a human axeman, we could reuse the ‘axe’ component. If we 
created a Kraken, we could reuse the ‘strong’ component.

## System Shock

The code above takes care of the entities and their components. But we still
need to handle them - how do we do that?

This is where things get interesting. You can have multiple systems to handle
the logic of all the entities that have a component from it. The physics system,
for example, would check for collisions for entities that have the ‘physics’ tag.

For this to be feasible, we want to add all entities to a list so that we can
iterate through them and do things like this:

    // Enemies only fight you if both are armed or unarmed
    // When at disadvantage, they run away
    for(var i = 0; i < entities.length; i++){
        if (player.armed && !entity[i].armed){
            entity[i].run();
        } else{
            entity[i].fight();
        }
    }

## Improving it 

One could argue that I didn’t break things down as much as I could - health
could be a component by itself as could others. You could then create component 
bundles for easy entity creation.

Another thing that could be improved is chaining components. For example:

    var ECS = function(){
      return { //We return the components
          health: function() {
            this.health= 100;
            return this; // We return the object
          },
          strong: function(){
              if(this.health !== null)
                  this.health = this.health*1.25;
              return this;
          }
      };
    };

This allows us a bit more flexibility and awesome chaining like so:

    var orc = new ECS();
    orc.health().strong(); // Clean and simple
    
    var orc2 = new ECS()
        .health()
        .strong(); // Even cleaner

## Real world examples

[CraftyJS][2] is a game framework built all around ECS and there are also
independent frameworks that one can adapt to game engines, like[ces.js][3] or
you could implement your own! A nice place to learn more is at the
[rectangle eater tutorial.][4]

I hope this is helpful, let me know in the comments what you think!

 [1]: img/ecs_orc.png
 [2]: http://invrse.co/entities-and-components-system/craftyjs.com/
 [3]: https://github.com/qiao/ces.js

 [4]: http://vasir.net/blog/game-development/how-to-build-entity-component-system-in-javascript