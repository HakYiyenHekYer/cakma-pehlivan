 const ctx = document.getElementById("canvas").getContext("2d");
GameEnded = false
ctx.translate(150,150)
ctx.scale(1,-1)
 ctx.font = "48px serif";
ctx.clear = function() {
  ctx.save()
  ctx.fillStyle = "#FFFFFF"
  ctx.fillRect(-150,-150,300,300)
  ctx.fillStyle = "#000000"
  ctx.strokeRect(-150,-150,300,300)
  ctx.restore()
}

function init() {
  window.requestAnimationFrame(main);
}

class Disc {
  constructor(x,y,r, color,id) {
    this.x = x
    this.y = y
    this.radius = r
    this.input = new Object()
    this.input.x = 0
    this.input.y = 0
    this.color = color
    this.id = id
  }
}
 var DiscList = new Array()
  var Player = new Disc(-75,0,10,"green",0)
  var Player2 = new Disc(75,0,10, "blue",1)
    DiscList.push(Player)
DiscList.push(Player2)
var FoodList = new Array()

function Dist(x1,x2,y1,y2) {
return (((x1-x2)**2)+((y1-y2)**2))**0.5  
}
ColorList = ["red", "#c1c100","#c1c101", "#d1d1a2"]

function spawnRates() {
  let RNG = Math.random()*100
  if (RNG >25) return ColorList[0]
  else return ColorList[1]
}

function foodSpawn() {
    if (FoodList.length < 5) {
      let Xsign = Math.random() > 0.5 ? -1 : 1
      let x = Math.random()*150*Xsign
      let Ysign = Math.random() > 0.5 ? -1 : 1
      let y = Math.random()*150*Ysign
      let radius = (Math.random()*4)+1
        let color = spawnRates()
      let newFood = new Disc(x,y,radius, color)
      FoodList.push(newFood)
    }
}

function yellowBullet(disc, target) {
  let newTarget = target == DiscList[0] ? DiscList[0] : DiscList[1]
  disc.color = ColorList[2]
  let dy = (target.y-disc.y)
  let dx = (target.x-disc.x)
    disc.input.x = 3*dx/(Math.abs(dx)+Math.abs(dy))
  disc.input.y = 3*dy/(Math.abs(dx)+Math.abs(dy))
  disc.target = newTarget
}


function orangeBullet(disc, target) {
  let newTarget = target == DiscList[0] ? DiscList[0] : DiscList[1]
  disc.color = ColorList[3]
  let dy = (target.y-disc.y)
  let dx = (target.x-disc.x)
    disc.input.x = 6*dx/(Math.abs(dx)+Math.abs(dy))
  disc.input.y = 6*dy/(Math.abs(dx)+Math.abs(dy))
  disc.target = newTarget
}

function eatFood() {
    for (let i=0; i<DiscList.length; i++) {
      let targetPlayer = DiscList[i]
  for (let j = 0; j<FoodList.length; j++) {
    let targetFood = FoodList[j]
    let calc = Dist(targetPlayer.x, targetFood.x, targetPlayer.y, targetFood.y)
    if ( calc < targetPlayer.radius) {
      if (targetFood.color == "red") {
      let newRadius = ((targetPlayer.radius**2)+(targetFood.radius**2))**0.5
      targetPlayer.radius = newRadius
        FoodList.splice(j, 1)
      }
      else if (targetFood.color == ColorList[1]) {
        let tgt = i == 1 ? Player : Player2
        yellowBullet(targetFood, tgt)
      }
      else if (targetFood.color == ColorList[2] && targetFood.target ==targetPlayer) {
        let tgt = i == 1 ? Player : Player2
        orangeBullet(targetFood, tgt)
      }
      else if (targetFood.color == ColorList[3] && targetFood.target ==targetPlayer) {
        targetFood.color = "#ffffff"
          targetPlayer.radius++
        targetFood.radius = 0
        FoodList.splice(j, 1)
        
      }
    }  
  }
    }
}

function deleteJunkFood() {
  for (let j = 0; j<FoodList.length; j++) {
    if (FoodList[j].x > 400 || FoodList[j].y > 400 ) FoodList.splice(j,1)
    j--
  }
}

function eatPlayer() {
  for (let i = 0; i< DiscList.length; i++) {
    let currentPlayer = DiscList[i]
    for (let j=i+1; j<DiscList.length; j++) {
    let targetPlayer = DiscList[j]
    let calc = Dist(currentPlayer.x, targetPlayer.x, currentPlayer.y, targetPlayer.y)
    if (calc < currentPlayer.radius || calc < targetPlayer.radius) {
      let biggerPlayer = currentPlayer.radius > targetPlayer.radius ? currentPlayer : targetPlayer
      let loserPlayer = currentPlayer.radius > targetPlayer.radius ? targetPlayer : currentPlayer 
      biggerPlayer.radius += loserPlayer.radius
      loserPlayer.radius = 0
      GameEnded = true
    }
    
    }
  }
}

addEventListener("keydown", (q) => {
    if (q.key == "ArrowRight") Player.input.x = 1
  else if (q.key == "ArrowLeft") Player.input.x = -1
  else if (q.key == "ArrowUp") Player.input.y = 1
  else if (q.key == "ArrowDown") Player.input.y = -1
  else if (q.key == "d") Player2.input.x = 1
  else if (q.key == "a") Player2.input.x = -1
  else if (q.key == "w") Player2.input.y = 1
  else if (q.key == "s") Player2.input.y = -1
})
addEventListener("keyup", (q) => {
    if (q.key == "ArrowRight") Player.input.x = 0
  else if (q.key == "ArrowLeft") Player.input.x = 0
  else if (q.key == "ArrowUp") Player.input.y = 0
  else if (q.key == "ArrowDown") Player.input.y = 0
  else if (q.key == "d") Player2.input.x = 0
  else if (q.key == "a") Player2.input.x = 0
  else if (q.key == "w") Player2.input.y = 0
  else if (q.key == "s") Player2.input.y = 0
})

function inputHandler() {

}

function update() {
  for (let i=0; i<DiscList.length; i++) {
    let target = DiscList[i]
      target.x += target.input.x
      target.y += target.input.y
  }
  for (let i=0; i<FoodList.length; i++) {
    let target = FoodList[i]
    target.x += target.input.x
      target.y += target.input.y
  }
    foodSpawn()
  eatFood()
  eatPlayer()
}

function render() {
  ctx.save()
  ctx.clear()
  if (GameEnded) {
ctx.save()
    ctx.scale(1,-1)
    ctx.fillText("Fecaat!", -75,-100)
    ctx.restore()
  } 
    for (let i = 0; i<DiscList.length; i++) {
      let Target = DiscList[i]
      ctx.save()
      ctx.beginPath()
      ctx.moveTo(Target.x, Target.y)
      ctx.arc(Target.x, Target.y, Target.radius, 2*Math.PI, false)
      ctx.fillStyle = Target.color
      ctx.fill()
      ctx.restore()
    }
  for (let i = 0; i<FoodList.length; i++) {
    ctx.save()
      let Target = FoodList[i]
      ctx.beginPath()
      ctx.moveTo(Target.x, Target.y)
      ctx.arc(Target.x, Target.y, Target.radius, 2*Math.PI, false)
      ctx.fillStyle = Target.color
      ctx.fill()
    ctx.restore()
    }
  ctx.restore()
}

function main() {
  window.requestAnimationFrame(main);
  inputHandler()
  update()
  render()
}

init()
