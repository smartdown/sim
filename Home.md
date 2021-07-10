## Home

This [Smartdown](https://smartdown.io) document ...

### P5JS for Simulation

```p5js /playable/autoplay

const scheduleTrackWidth = 50;
const margin = 40;
const res = 20;
const div = this.div;

var locs = [];
function calcVec(x, y) {
  return new P5.Vector(y - x, - x - y);
}

p5.windowResized = function() {
  p5.resizeCanvas(div.clientWidth, 500);

  var countX = p5.ceil((p5.width - scheduleTrackWidth) / res) + 1;
  var countY = p5.ceil(p5.height / res) + 1;

  for (var j = 0; j < countY; j++) {
    for (var i = 0; i < countX; i++) {
      locs.push(new P5.Vector(res * i, res * j));
    }
  };
};

p5.setup = function() {
  p5.createCanvas(100, 100);
  p5.windowResized();

  p5.noFill();
  p5.stroke(249, 78, 128);
};

function drawScheduleTrack() {
  p5.fill('white');

  const height = p5.height;
  p5.rect(0, 0, scheduleTrackWidth, p5.height);
  p5.textSize(10);
  p5.fill('red');

  let week = 1;
  for (let y = 0; y < height; y += 20) {
    p5.text(`Week ${week}`, 2, y);
    ++week;
  }
}

p5.draw = function() {
  p5.background(30, 67, 137);
  drawScheduleTrack();

  for (var i = locs.length - 1; i >= 0; i--) {
    var h = calcVec(locs[i].x - p5.mouseX, locs[i].y - p5.mouseY);
    p5.push();
      p5.translate(locs[i].x + scheduleTrackWidth + res, locs[i].y + res);
      p5.rotate(h.heading());
      p5.line(0, 0, 0, - 15);
    p5.pop();
  };
};
```

### Exploring `js-simulator`

- [js-simulator](https://github.com/chen0040/js-simulator#readme)

https://unpkg.com/js-simulator@1.0.17/build/jssim.min.js

```javascript /playable
//smartdown.import=https://unpkg.com/js-simulator/build/jssim.min.js

this.div.innerHTML = `
<h2>Discrete-Event Simulator: Virus <input type="text" id="simTime" value="" /></h2>

<canvas id="myCanvas" width="800" height="600" style="border:1px solid #000000;">
</canvas>
`;
var XMIN = 0;
var XMAX = 800;
var YMIN = 0;
var YMAX = 600;

var DIAMETER = 8;

var HEALING_DISTANCE = 20;
var HEALING_DISTANCE_SQUARED = HEALING_DISTANCE * HEALING_DISTANCE;
var INFECTION_DISTANCE = 20;
var INFECTION_DISTANCE_SQUARED = INFECTION_DISTANCE * INFECTION_DISTANCE;

var NUM_HUMANS = 100;
var NUM_GOODS = 4;
var NUM_EVILS = 4;

const cdnPrefix = 'https://cdn.jsdelivr.net/gh/chen0040/js-simulator@master/examples/';
var healthy_icon = new Image();
healthy_icon.src = cdnPrefix + "normal.png";
var infected_icon = new Image();
infected_icon.src = cdnPrefix + "infected.png";
var good_icon = new Image();
good_icon.src = cdnPrefix + "good.png";
var evil_icon = new Image();
evil_icon.src = cdnPrefix + "virus.png";

var Evil = function(id, loc, space) {
    jssim.SimEvent.call(this);
    this.id = id;
    this.agentLocation = loc;
    this.color = '#ff0000';
    this.desiredLocation = null;
    this.suggestedLocation = null;
    this.steps = 0;  
    this.space = space;
    space.updateAgent(this, loc.x, loc.y);
    this.type = 'Evil';
    this.greedy = false;
};

Evil.prototype = Object.create(jssim.SimEvent.prototype);

Evil.prototype.update = function(deltaTime) {
    var mysteriousObjects = this.space.getNeighborsWithinDistance(this.agentLocation, 10.0 * INFECTION_DISTANCE);
    
    var distance2DesiredLocation = 1000000;
    for(var i = 0 ; i < mysteriousObjects.length ; i++ )
    {
        if(mysteriousObjects[i] != this )
        {
            if(mysteriousObjects[i].type != 'Human') continue;
            var ta = mysteriousObjects[i];
            if(ta.isInfected()) continue;
            if(withinInfectionDistance(this.agentLocation, ta.agentLocation))
                ta.setInfected( true );
            else
            {
                if(this.greedy)
                {
                    var tmpDist = distanceSquared(this.agentLocation, ta.agentLocation);
                    if(tmpDist <  distance2DesiredLocation )
                    {
                        this.desiredLocation = ta.agentLocation;
                        distance2DesiredLocation = tmpDist;
                    }
                }
            }
        }
    }
        

    this.steps--;
    if( this.desiredLocation == null || !this.greedy )
    {
        if(this.steps <= 0 )
        {
            this.suggestedLocation = new jssim.Vector2D((Math.random()-0.5)*((XMAX-XMIN)/5-DIAMETER) + this.agentLocation.x,
                (Math.random()-0.5)*((YMAX-YMIN)/5-DIAMETER) + this.agentLocation.y);
            this.steps = 100;
        }
        this.desiredLocation = this.suggestedLocation;
    }

    var dx = this.desiredLocation.x - this.agentLocation.x;
    var dy = this.desiredLocation.y - this.agentLocation.y;

    var temp = 0.5 * Math.sqrt(dx*dx+dy*dy);
    if( temp < 1 )
    {
        this.steps = 0;
    }
    else
    {
        dx /= temp;
        dy /= temp;
    }
                

    if( !acceptablePosition(this, new jssim.Vector2D(this.agentLocation.x + dx, this.agentLocation.y + dy), this.space) )
    {
        this.steps = 0;
    }
    else
    {
        this.agentLocation = new jssim.Vector2D(this.agentLocation.x + dx, this.agentLocation.y + dy);
        space.updateAgent(this, this.agentLocation.x, this.agentLocation.y);
    }
};

Evil.prototype.draw = function(context, pos) {
    context.drawImage(evil_icon, pos.x, pos.y);    
};

var Good = function(id, loc, space) {
    jssim.SimEvent.call(this);
    this.id = id;
    this.agentLocation = loc;
    this.color = '#00ff00';
    this.desiredLocation = null;
    this.suggestedLocation = null;
    this.steps = 0;  
    this.space = space;
    space.updateAgent(this, loc.x, loc.y);
    this.type = 'Good';
    this.greedy = true;
};

Good.prototype = Object.create(jssim.SimEvent.prototype);

Good.prototype.update = function(deltaTime) {
    var mysteriousObjects = this.space.getNeighborsWithinDistance(this.agentLocation, 10.0 * INFECTION_DISTANCE);
    
    var distance2DesiredLocation = 1000000;
    for(var i = 0 ; i < mysteriousObjects.length ; i++ )
    {
        if(mysteriousObjects[i] != this )
        {
            if(mysteriousObjects[i].type != 'Human') continue;
            var ta = mysteriousObjects[i];
            if(!ta.isInfected()) continue;
            if(withinHealingDistance(this.agentLocation, ta.agentLocation))
                ta.setInfected(false);
            else
            {
                if(this.greedy)
                {
                    var tmpDist = distanceSquared(this.agentLocation, ta.agentLocation);
                    if(tmpDist <  distance2DesiredLocation )
                    {
                        this.desiredLocation = ta.agentLocation;
                        distance2DesiredLocation = tmpDist;
                    }
                }
            }
        }
    }
        

    this.steps--;
    if( this.desiredLocation == null || !this.greedy )
    {
        if(this.steps <= 0 )
        {
            this.suggestedLocation = new jssim.Vector2D((Math.random()-0.5)*((XMAX-XMIN)/5-DIAMETER) + this.agentLocation.x,
                (Math.random()-0.5)*((YMAX-YMIN)/5-DIAMETER) + this.agentLocation.y);
            this.steps = 100;
        }
        this.desiredLocation = this.suggestedLocation;
    }

    var dx = this.desiredLocation.x - this.agentLocation.x;
    var dy = this.desiredLocation.y - this.agentLocation.y;

    var temp = 0.5 * Math.sqrt(dx*dx+dy*dy);
    if( temp < 1 )
    {
        this.steps = 0;
    }
    else
    {
        dx /= temp;
        dy /= temp;
    }
                

    if( !acceptablePosition(this, new jssim.Vector2D(this.agentLocation.x + dx, this.agentLocation.y + dy), this.space))
    {
        this.steps = 0;
    }
    else
    {
        this.agentLocation = new jssim.Vector2D(this.agentLocation.x + dx, this.agentLocation.y + dy);
        space.updateAgent(this, this.agentLocation.x, this.agentLocation.y);
    }
};

Good.prototype.draw = function(context, pos) {
    context.drawImage(good_icon, pos.x, pos.y);  
};

var Human = function(id, loc, space) {
    jssim.SimEvent.call(this);
    this.id = id;
    this.agentLocation = loc;
    this.color = '#ff8800';
    this.desiredLocation = null;
    this.suggestedLocation = null;
    this.steps = 0;  
    this.space = space;
    space.updateAgent(this, loc.x, loc.y);
    this.type = 'Human';
    this.infected = false;
};

Human.prototype = Object.create(jssim.SimEvent.prototype);

Human.prototype.setInfected = function(infected) {
    this.infected = infected;
    if(infected){
        this.color = '#5533ff';
    } else {
        this.color = '#ff8800';
    }
};

Human.prototype.isInfected = function() {
    return this.infected;
};

Human.prototype.update = function(deltaTime) {
    this.steps--;
    if( this.desiredLocation == null || this.steps <= 0 )
    {
        this.desiredLocation = new jssim.Vector2D((Math.random()-0.5)*((XMAX-XMIN)/5-DIAMETER) + this.agentLocation.x,
            (Math.random()-0.5)*((YMAX-YMIN)/5-DIAMETER) + this.agentLocation.y);
        this.steps = 50 + Math.floor(Math.random() * 50);
    }

    var dx = this.desiredLocation.x - this.agentLocation.x;
    var dy = this.desiredLocation.y - this.agentLocation.y;


    var temp = Math.sqrt(dx*dx+dy*dy);
    if( temp < 1 )
    {
        this.steps = 0;
    }
    else
    {
        dx /= temp;
        dy /= temp;
    }


    if( ! acceptablePosition(this, new jssim.Vector2D(this.agentLocation.x + dx, this.agentLocation.y + dy ), this.space) )
    {
        steps = 0;
    }
    else
    {
        this.agentLocation = new jssim.Vector2D(this.agentLocation.x + dx, this.agentLocation.y + dy);
        this.space.updateAgent(this, this.agentLocation.x, this.agentLocation.y);
    }  
};

Human.prototype.draw = function(context, pos) {
    if(this.infected) {
        context.drawImage(infected_icon, pos.x, pos.y);
    }  else {
        context.drawImage(healthy_icon, pos.x, pos.y);
    }
};



function distanceSquared(loc1, loc2)
{
    return( (loc1.x-loc2.x)*(loc1.x-loc2.x)+(loc1.y-loc2.y)*(loc1.y-loc2.y) );
}

function conflict(a, b)
{
    if( ( ( a.x > b.x && a.x < b.x+DIAMETER ) ||
            ( a.x+DIAMETER > b.x && a.x+DIAMETER < b.x+DIAMETER ) ) &&
            ( ( a.y > b.y && a.y < b.y+DIAMETER ) ||
            ( a.y+DIAMETER > b.y && a.y+DIAMETER < b.y+DIAMETER ) ) )
    {
        return true;
    }
    return false;
}

function withinInfectionDistance(a, b)
{
    return ( (a.x-b.x)*(a.x-b.x)+(a.y-b.y)*(a.y-b.y) <= INFECTION_DISTANCE_SQUARED );
}

function withinHealingDistance(a, b )
{
    return ( (a.x-b.x)*(a.x-b.x)+(a.y-b.y)*(a.y-b.y) <= HEALING_DISTANCE_SQUARED );
}

function acceptablePosition(agent, location, space)
{
    if( location.x < DIAMETER/2 || location.x > (XMAX-XMIN)-DIAMETER/2 ||
        location.y < DIAMETER/2 || location.y > (YMAX-YMIN)-DIAMETER/2 )
        return false;
    var mysteriousObjects = space.getNeighborsWithinDistance( location, 2*DIAMETER );
    for(var i = 0 ; i < mysteriousObjects.length ; i++ )
    {
        if(mysteriousObjects[i] != agent)
        {
            var ta = mysteriousObjects[i];
            if(conflict(location, space.getLocation(ta.id))) return false;
        }
    }
    return true;
}


var scheduler = new jssim.Scheduler();

var space = new jssim.Space2D();


function reset() {
    scheduler.reset(); 
    space.reset();
    
    for(var x=0;x<NUM_HUMANS+NUM_GOODS+NUM_EVILS;x++) {
        var dx = Math.floor(Math.random() * 10) - 5;
        var dy = Math.floor(Math.random() * 10) - 5;
        
        var loc = null;
        var agent = null;
        var times = 0;
        while(loc == null || !acceptablePosition(agent, loc, space))
        {
            loc = new jssim.Vector2D(Math.random()*(XMAX-XMIN-DIAMETER)+XMIN+DIAMETER/2,
                Math.random()*(YMAX-YMIN-DIAMETER)+YMIN+DIAMETER/2 );
            if( x < NUM_HUMANS )
                agent = new Human( "Human"+x, loc, space);
            else if( x < NUM_HUMANS+NUM_GOODS ) {
                agent = new Good( "Good"+(x-NUM_HUMANS), loc, space);
                agent.greedy = Math.random() < 0.5;
            }
            else {
                agent = new Evil( "Evil"+(x-NUM_HUMANS-NUM_GOODS), loc, space);
                agent.greedy = Math.random() < 0.5;
            }
            times++;
            if( times == 1000 )
            {
                break;
            }
        };
        scheduler.scheduleRepeatingIn(agent, 1);
    }
}

reset();

var canvas = document.getElementById("myCanvas");

setInterval(function(){ 
    if(scheduler.current_time == 10000) {
        reset();
    }
    scheduler.update();
    space.render(canvas);
    //console.log('current simulation time: ' + scheduler.current_time);
    document.getElementById("simTime").value = "Simulation Time: " + scheduler.current_time;
}, 50);

```

### Virus Simulation

Based on [example-virus](https://github.com/chen0040/js-simulator/blob/master/examples/example-virus.html)

#### Description

A simulation of intentional virus infection and disinfection in a population.  The bad guys (red) either move randomly or greedily chase after the nearest uninfected individual (in orange).  Infected individuals (in purple) move randomly and may be disinfected by good guys (in green) who likewise either move randomly or greedily.


