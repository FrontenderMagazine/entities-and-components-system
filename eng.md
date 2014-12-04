<article class="post-content">
Imagine you are creating a roleplaying game. You can choose between 4 playable
characters that differ greatly between themselves - it should be simple enough 
to implement it in a classical way (or even as simple objects). For example:

    <span class="kd">var</span> <span class="nx">Hero</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">health</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">power</span> <span class="o">=</span> <span class="mi">20</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">sprite</span> <span class="o">=</span> <span class="s1">'hero.png'</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">weapon</span> <span class="o">=</span> <span class="s1">'spear'</span><span class="p">;</span>
        <span class="k">this</span><span class="p">.</span><span class="nx">damage</span> <span class="o">=</span> <span class="mi">20</span><span class="p">;</span>
    <span class="p">};</span>
    
    <span class="kd">var</span> <span class="nx">hero1</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Hero</span><span class="p">();</span>

### No Country for Old Orcs {#no-country-for-old-orcs}

Cool, that works. It doesn’t feel ideal though. What if we add new characters
later? And what about the enemies?

Let’s forget the extra character for now (leave it to a DLC) and let’s
focus on the enemies. Suppose we want our orcs to be generated at random (yay 
diversity) from a set of attributes - maybe we want some that are strong and 
some that are weak, maybe some with bows, others with axes.

An awesome way to approach this problem is by using an entities and components
system.

![Avengers, assemble!][1]

### Yo Dawg I heard you like components {#yo-dawg-i-heard-you-like-components
}

Think of components as the smallest building blocks that, assembled together,
form an entity. For our orc-folk, maybe we could do this:

    <span class="kd">var</span> <span class="nx">components</span> <span class="o">=</span> <span class="p">{</span>
      <span class="nx">orc</span><span class="o">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">entity</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">health</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">humanoid</span> <span class="o">=</span> <span class="kc">true</span><span class="p">;</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">color</span> <span class="o">=</span> <span class="s1">'green'</span><span class="p">;</span>
      <span class="p">},</span>
    
      <span class="nx">strong</span><span class="o">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">entity</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">if</span><span class="p">(</span><span class="nx">entity</span><span class="p">.</span><span class="nx">health</span> <span class="o">!==</span> <span class="kc">null</span><span class="p">)</span>
          <span class="nx">entity</span><span class="p">.</span><span class="nx">health</span> <span class="o">=</span> <span class="nx">entity</span><span class="p">.</span><span class="nx">health</span><span class="o">*</span><span class="mf">1.25</span><span class="p">;</span>
      <span class="p">},</span>
      
      <span class="nx">axe</span><span class="o">:</span> <span class="kd">function</span><span class="p">(</span><span class="nx">entity</span><span class="p">)</span> <span class="p">{</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">weapon</span> <span class="o">=</span> <span class="s1">'Axe'</span><span class="p">;</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">damage</span> <span class="o">=</span> <span class="mi">5</span><span class="p">;</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">critMulti</span> <span class="o">=</span> <span class="mi">4</span><span class="p">;</span>
        <span class="nx">entity</span><span class="p">.</span><span class="nx">critChance</span> <span class="o">=</span> <span class="mi">20</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">};</span>

Then if we do something like this:

    <span class="kd">var</span> <span class="nx">orc1</span> <span class="o">=</span> <span class="p">{};</span>
    <span class="nx">components</span><span class="p">.</span><span class="nx">orc</span><span class="p">(</span><span class="nx">orc1</span><span class="p">);</span>
    <span class="nx">components</span><span class="p">.</span><span class="nx">strong</span><span class="p">(</span><span class="nx">orc1</span><span class="p">);</span>
    <span class="nx">components</span><span class="p">.</span><span class="nx">axe</span><span class="p">(</span><span class="nx">orc1</span><span class="p">);</span>

We will have an armed, tough orc ready! How cool and simple is this? If we
wanted to create a human axeman, we could reuse the ‘axe’ component. If we 
created a Kraken, we could reuse the ‘strong’ component.

### System Shock {#system-shock}

The code above takes care of the entities and their components. But we still
need to handle them - how do we do that?

This is where things get interesting. You can have multiple systems to handle
the logic of all the entities that have a component from it. The physics system,
for example, would check for collisions for entities that have the ‘physics’ tag.

For this to be feasible, we want to add all entities to a list so that we can
iterate through them and do things like this:

    <span class="c1">//Enemies only fight you if both are armed or unarmed</span>
    <span class="c1">//When at disadvantage, they run away</span>
    <span class="k">for</span> <span class="p">(</span><span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o"><</span> <span class="nx">entities</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="nx">player</span><span class="p">.</span><span class="nx">armed</span> <span class="o">&&</span> <span class="o">!</span><span class="nx">entity</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">armed</span><span class="p">)</span> <span class="p">{</span>
            <span class="nx">entity</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">run</span><span class="p">();</span>
        <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
            <span class="nx">entity</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">fight</span><span class="p">();</span>
        <span class="p">}</span>
    <span class="p">}</span>

### Improving it {#improving-it}

One could argue that I didn’t break things down as much as I could - health
could be a component by itself as could others. You could then create component 
bundles for easy entity creation.

Another thing that could be improved is chaining components. For example:

    <span class="kd">var</span> <span class="nx">ECS</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
      <span class="k">return</span> <span class="p">{</span> <span class="c1">//We return the components</span>
          <span class="nx">health</span><span class="o">:</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
            <span class="k">this</span><span class="p">.</span><span class="nx">health</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>
            <span class="k">return</span> <span class="k">this</span><span class="p">;</span> <span class="c1">//We return the object</span>
          <span class="p">},</span>
          <span class="nx">strong</span><span class="o">:</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
              <span class="k">if</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">health</span> <span class="o">!==</span> <span class="kc">null</span><span class="p">)</span>
                  <span class="k">this</span><span class="p">.</span><span class="nx">health</span> <span class="o">=</span> <span class="k">this</span><span class="p">.</span><span class="nx">health</span><span class="o">*</span><span class="mf">1.25</span><span class="p">;</span>
              <span class="k">return</span> <span class="k">this</span><span class="p">;</span>
          <span class="p">}</span>
      <span class="p">};</span>
    <span class="p">};</span>

This allows us a bit more flexibility and awesome chaining like so:

    <span class="kd">var</span> <span class="nx">orc</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">ECS</span><span class="p">();</span>
    <span class="nx">orc</span><span class="p">.</span><span class="nx">health</span><span class="p">().</span><span class="nx">strong</span><span class="p">();</span> <span class="c1">//Clean and simple</span>
    
    <span class="kd">var</span> <span class="nx">orc2</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">ECS</span><span class="p">()</span>
        <span class="p">.</span><span class="nx">health</span><span class="p">()</span>
        <span class="p">.</span><span class="nx">strong</span><span class="p">();</span> <span class="c1">//Even cleaner</span>

### Real world examples {#real-world-examples}

[CraftyJS][2] is a game framework built all around ECS and there are also
independent frameworks that one can adapt to game engines, like[ces.js][3] or
you could implement your own! A nice place to learn more is at the
[rectangle eater tutorial.][4]

I hope this is helpful, let me know in the comments what you think!</article>

 [1]: img/ecs_orc.png
 [2]: http://invrse.co/entities-and-components-system/craftyjs.com/
 [3]: https://github.com/qiao/ces.js

 [4]: http://vasir.net/blog/game-development/how-to-build-entity-component-system-in-javascript