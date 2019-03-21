# 中介者模式

中介者模式的作用就是解除对象与对象之间的紧耦合关系。增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需要通知中介者对象即可。



#### 用中介者模式实现泡泡堂游戏

```js
function Player( name, teamColor ) {
    this.name = name; // 角色名字
    this.teamColor = teamColor; // 队伍颜色
    this.state = 'alive'; // 玩家生存状态
}

Player.prototype.win = function() {
    console.log(this.name + ' won');
};

Player.prototype.lose = function() {
    console.log(this.name + ' lost');
}

/***************玩家死亡****************/

Player.prototype.die = function() {
    this.state = 'dead';
    playerDirector.ReceiveMessage('playerDead', this); // 给中介者发送消息，玩家死亡
};

/***************移除玩家****************/
Player.prototype.remove = function() {
    playerDirector.ReceiveMessage('removePlayer', this); // 给中介者发送消息，移除一个玩家
}

/***************玩家换队****************/
Player.prototype.changeTeam = function() {
    playerDirector.ReceiveMessage( 'changeTeam', this, color); // 给中介者发送消息，玩家换队
}

// 创建玩家的工厂函数
var playerFactory = function(name, teamColor) {
    var newPlayer = new Player(name, teamColor); // 创造一个新的玩家对象
    playerDirector.ReceiveMessage( 'addPlayer', newPlayer); // 给中介者发送消息，新增玩家
    
    return new Player;
};
```

最后，我们需要实现这个中介者`playerDirector`对象，一般有一下两种方式：

- 利用发布-订阅模式，将 `playerDirector` 实现为订阅者，各 `player` 作为发布者，一旦 `palyer` 的状态发送改变，便推送消息给 `palyerDirector`， `playerDirector`处理消息后将反馈发送给气体 `palyer` 。
- 在 `playerDirector` 中开放一些接收消息的接口， 各 `palyer` 可以直接调用该接口来给 `playerDirector` 发送消息，`palyer` 只需要传递一个参数给 `palyerDirector`， 这个参数的目的是使 `palyerDirector` 可以识别发送者。 同样，`palyerDirector` 接收到消息之后会将处理结果反馈给其他 `palyer` 。



这里我们使用第二种方式，`palyerDirector` 开放一个对外暴露的接口 `ReceiveMessage`， 负责接收 `palyer` 对象发送的消息，而 player 对象发送消息的时候， 总是把自身 this 作为参数发送给 `palyerDirector`， 以便 `playerDirector` 识别消息来自于哪个玩家对象。

```js
var playerDirector = ( function() {
	var players = {}, // 保存所有玩家
        operations = {}; // 中介者可以执行的操作
    
    /***************新增一个玩家****************/
    operations.addPlayer = function( palyer ) {
        var teamColor = palyer.teamColor; // 玩家的队伍颜色
        // 如果该颜色的玩家还没有成立队伍，则新成立一个队伍
        palyers[ teamColor ] = palyers[ teamColor ] || [];
        
        palyers[teamColor].push( palyer ); // 添加玩家进队伍
     };
    
    /***************移除一个玩家****************/
    operations.removePlayer = function( palyer ) {
        var teamColor = player.teamColor, // 玩家的队伍颜色
            teamPlayers = players[ teamColor ] || []; // 该队伍所有成员
        for (var i = teamPlayers.length - 1; i >= 0; i-- ) { // 遍历删除
            if ( teamPlayers[i] === palyer ) {
                teamPlayers.splice(i, 1);
            }
        }
    };
    
    /***************玩家换队****************/
    operations.changeTeam = function( palyer, newTeamColor) {
        operations.removePlayer( palyer ); // 从原队伍中删除
        palyer.teamColor = newTeamColor; //改变队伍颜色
        operations.addPlayer(player);	// 增加到新队伍中
    }
    
    /***************玩家死亡****************/
    operations.playerDead = function( player ) {
        var teamColor = player.teamColor,
            teamPlayers = players[ teamColor ]; // 玩家所在队伍
        
        var all_dead = true;
        
        for (var i = 0, player; player = teamPlayers[i++]; ) {
            if ( player.state !== 'dead') {
                all_dead = false;
                break;
            }
        }
        
        if (all_dead === true) {
            for ( var i = 0, player; player = teamPlayers[i++]; ) {
                player.lose(); // 本队所有玩家lose
            }
            
            for ( var color in players) {
                if ( color !== teamColor ) {
                    var teamPlayers = players[color]; // 其他队伍的玩家
                    for ( var i = 0, player; player = teamPlayers[i++]; ) {
                        player.win(); // 其他队伍所有玩家win
                    }
                }
            }
        }
    };
    
    var ReceiveMessage = function() {
        var message = Array.prototype.shift.call( arguments ); // arguments的第一个参数为消息名称
        operations[message].apply(this, arguments);
    };
    
    return {
		ReceiveMessage: ReceiveMessage
    }
})();
```

测试

```js
// 红队
var player1 = playerFactory('皮蛋', 'red');
var player2 = playerFactory('小乖', 'red');
var player3 = playerFactory('宝宝', 'red');
var player4 = playerFactory('小强', 'red');

// 蓝队
var player5 = playerFactory('黑妞', 'blue');
var player6 = playerFactory('葱头', 'blue');
var player7 = playerFactory('胖墩', 'blue');
var player8 = playerFactory('海盗', 'blue');

// 红队都死亡
player1.die();
player2.die();
player3.die();
player4.die();

// 假设皮蛋和小乖掉线
player1.remove();
player2.remove();
player3.die();
player4.die();

// 假设皮蛋从红队叛变到蓝队
player1.changeTeam('blue');
player2.die();
player3.die();
player4.die();
```

可以看到，除了中介者本身，没有任何一个玩家直到其他任何玩家的存在，玩家与玩家之间的耦合关系已经完全解除，某个玩家的任何操作都不需要通知其他玩家，而只需要给中介者发送一个消息，中介者处理完消息之后会把处理结果反馈给其他的玩家对象。