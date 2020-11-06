title: 《JavaScript 设计模式与开发实战》读书笔记 14
date: 2017-04-15 13:56:47

---

第十四章 中介者模式
<!-- more -->

在程序里，也许一个对象会保持对其他多个对象的引用，当对象很多，难免会形成网状的交叉引用。当改变或删除其中一个对象时，需要通知所有引用到它的对象。

中介者模式的作用就是解除对象与对象之间的紧耦合关系。增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需通知中介者对象即可。中介者模式使网状的多对多关系变成了相对简单的一对多关系。

## 14.2 中介者模式的例子——泡泡堂游戏

首先是定义 `Player` 构造函数和 `player` 对象的原型方法，在 `player` 对象的这些原型方法中，不再负责具体的执行逻辑，而是把操作转交给中介者对象 `playerDirector`。

```javascript
function Player(name, teamColor) {
    this.name = name
    this.teamColor = teamColor
    this.state = 'alive'
}

Player.prototype.win = function() {
    console.log(this.name + 'won')
}

Player.prototype.lose = function() {
    console.log(this.name + 'lost')
}

Player.prototype.die = function() {
    this.state = 'dead'
    playerDirector.ReceiveMessage('playerDead', this)
}

Player.prototype.remove = function() {
    playerDirector.ReceiveMessage('removePlayer', this)
}

Player.prototype.changeTeam = function() {
    playerDirector.ReceiveMessage('changeTeam', this, color)
}
```

工厂函数里不再需要给创建的玩家对象设置队友和敌人，这个工厂函数几乎失去了工厂的意义

```javascript
var playerFactory = function(name, teamColor) {
    var newPlayer = new Player(name, teamColor)
    playerDirector.ReceiveMessage('addPlayer', newPlayer)

    return newPlayer
}
```

实现中介者 `playerDirector` 对象一般有两种方式

- 发布-订阅模式。将 `playerDirector` 实现为订阅者，各 `player` 作为发布者。
- 在 `playerDirector`中开放一些接收消息的接口，各 `player` 可以直接调用该接口来给 `playerDirector` 发送消息，`player` 只需传递一个参数给 `playerDirector`，这个参数的目的是使 `playerDirector` 可以识别发送者。同样，`playerDirector` 接收到消息之后会将处理结果反馈给其他 `player`。

这里我们使用第二种方式，`playerDirector` 开放一个对外暴露的接口 `ReceiveMessage`，负责接收 `player` 对象发送的消息，而 `player` 对象发送消息的时候，总是把自身 `this` 作为参数发送给 `playerDirector`，以便 `playerDirector` 识别。

```javascript
var playerDirector = (function() {
    var players = {},
        operations = {}

    operations.addPlayer = function(player) {
        var teamColor = player.teamColor
        players[teamColor] = players[teamColor] || []
        players[teamColor].push(player)
    }

    operations.removePlayer = function(player) {
        var teamColor = player.teamColor,
            teamPlayers = players[teamColor] || []
        for (var i = teamPlayers.length - 1; i >= 0; i--) {
            if (teamPlayers[i] === player) {
                teamPlayers.splice(i, 1)
            }
        }
    }

    operations.changeTeam = function(player, newTeamColor) {
        operations.removePlayer(player)
        player.teamColor = newTeamColor
        operations.addPlayer(player)
    }

    operations.playerDead = function(player) {
        var teamColor = player.teamColor,
            teamPlayers = players[teamColor]

        var all_dead = true

        for (var i = 0, player; player = teamPlayers[i++];) {
            if (player.state !== 'dead') {
                all_dead = false
                break
            }
        }

        if (all_dead === true) {
            for (var i = 0, player; player = teamPlayers[i++];) {
                player.lose()
            }

            for (var color in players) {
                if (color !== teamColor) {
                    var teamPlayers = players[color]
                    for (var i = 0, player; player = teamPlayers[i++];) {
                        player.win()
                    }
                }
            }
        }
    }

    var ReceiveMessage = function() {
        var message = Array.prototype.shift.call(arguments)
        operations[message].apply(this, arguments)
    }

    return {
        ReceiveMessage: ReceiveMessage
    }
})();
```

除了中介者本身，没有一个玩家知道其他任何玩家的存在，玩家与玩家之间的耦合关系已经完全解除，某个玩家的任何操作都不需要通知其他玩家，而只需要给中介者发送一个消息，中介者处理完消息之后会把处理结果反馈给其他的玩家对象。我们还可以给中介者扩展更多功能。

## 14.4 小结

中介者模式是迎合迪米特法则的一种实现。迪米特法则也叫最少知识原则，是指一个对象应该尽可能少地了解另外的对象。如果对象之间的耦合性太高，一个对象发生改变之后，难免会影响到其他对象。而在中介者模式中，对象之间几乎不知道彼此的存在，只能通过中介者对象来相互影响对方。

中介者模式使各个对象之间得以解耦，以中介者和对象之间的一对多关系取代了对象之间的网状多对多关系。各个对象只需关注自身功能的实现，对象之间的交互关系交给了中介者对象来实现和维护。

中介者模式最大的缺点是系统中会增加一个中介者对象，因为对象之间
