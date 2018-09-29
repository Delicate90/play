<template>
  <div class="container">
    <div class="process-box">
      <div class="process-item process-player" :style="{bottom: (player.curSkillTime - 15)+'px'}">
        {{number(player.curSkillTime)}}
      </div>
      <div class="process-item process-npc" :style="{bottom: (npc.curSkillTime - 15)+'px'}">
        {{number(npc.curSkillTime)}}
      </div>
    </div>
    <div class="text-box">
      <div class="info-box">
        <div class="info-item">
          <div class="info-hp">
            <div class="info-hp-inner" :style="{
              width: (player.hp/player.hpMax)*100+'%',
              backgroundColor: bloodColor((player.hp/player.hpMax)*100)
            }"></div>
            <div class="info-hp-num">{{player.hp}} / {{player.hpMax}}</div>
          </div>
          <div class="info-bottom-box">
            <div class="info-name">{{player.name}}</div>
            <div class="info-skills">
              <ul class="info-skills-list">

              </ul>
            </div>
          </div>

          <div class="info-skill">{{player.skill.name}} / {{player.skill.time}}s / hp-{{player.skill.damage}}</div>
        </div>
        <div class="info-item">
          <div class="info-name">{{npc.name}} {{npc.hp}}/{{npc.hpMax}}</div>
          <div class="info-skill">{{npc.skill.name}} / {{npc.skill.time}}s / hp-{{npc.skill.damage}}</div>
        </div>
      </div>
      <div class="record-box" ref="record_box">
        <ul class="record-list">
          <li class="record-item" v-for="record in records" v-html="record"></li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import Moment from 'moment'

  export default {
    name: 'HelloWorld',
    data() {
      return {
        player: {
          isPlayer: true,
          name: '龙傲天',
          head: '',
          hp: 200,
          hpMax: 200,
          curSkillTime: 0,
          curSkill: {},
          skills: [
            {
              name: '攻击',
              time: 2,
              damage: 5
            }, {
              name: '火球术',
              time: 4,
              damage: 13
            }, {
              name: '御剑术',
              time: 3,
              damage: 10
            }
          ]
        },
        npc: {
          isPlayer: false,
          name: '稻草人',
          head: '',
          hp: 135,
          hpMax: 135,
          curSkillTime: 0,
          curSkill: {},
          skills: [
            {
              name: '攻击',
              time: 2,
              damage: 15
            }, {
              name: '鞭挞',
              time: 3,
              damage: 18
            }, {
              name: '猛击',
              time: 5,
              damage: 19
            }
          ]
        },
        records: []
      }
    },
    computed: {},
    methods: {
      reduce() {
        --this.player.curSkillTime;
        --this.npc.curSkillTime;
      },
      timer() {
        this.reduce();
        if (this.player.curSkillTime === 0 || this.npc.curSkillTime === 0) {
          this.trigger()
        } else {
          window.requestAnimationFrame(this.timer)
        }
      },
      number(num) {
        return Math.ceil(num / 60)
      },
      bloodColor(num) {
        if (num > 60) {
          return 'yellowgreen';
        } else if (num > 30) {
          return 'gold';
        } else {
          return 'crimson';
        }
      },
      trigger() {
        if (this.player.curSkillTime === 0 && this.npc.curSkillTime === 0) {
          this.attack(this.player, this.npc);
          this.attack(this.npc, this.player);
          if (this.npc.hp > 0 && this.player.hp > 0) {
            this.chooseSkill(this.player);
            this.chooseSkill(this.npc);
            this.timer();
          } else {
            this.result();
            return;
          }
        }
        if (this.player.curSkillTime === 0) {
          this.attack(this.player, this.npc);
          if (this.npc.hp > 0 && this.player.hp > 0) {
            this.chooseSkill(this.player);
            this.timer();
          } else {
            this.result();
            return;
          }
        }
        if (this.npc.curSkillTime === 0) {
          this.attack(this.npc, this.player);
          if (this.npc.hp > 0 && this.player.hp > 0) {
            this.chooseSkill(this.npc);
            this.timer();
          } else {
            this.result();
            return;
          }
        }
      },
      chooseSkill(item) {
        const skill = item.skills[Math.floor(Math.random() * item.skills.length)];
        item.curSkillTime = skill.time * 60;
        item.skill = skill;
      },
      attack(a, b) {
        if (a.hp>0 && b.hp > 0) {
          b.hp = b.hp - a.skill.damage < 0 ? 0 : b.hp - a.skill.damage;
          const time = Moment().format('YYYY-MM-DD HH:mm:ss');
          this.output(`[${time}] "<span class="${a.isPlayer ? 'record-item-player' : 'record-item-npc'}">${a.name}</span>"对"<span class="${a.isPlayer ? 'record-item-npc' : 'record-item-player'}">${b.name}</span>"使用了 [<span class="record-item-skill">${a.skill.name}</span>],造成了<span class="record-item-damage">${a.skill.damage}</span>点伤害。`);
        }
      },
      loot() {

      },
      result() {
        const time = Moment().format('YYYY-MM-DD HH:mm:ss');
        if (this.player.hp > 0) {
          this.output(`[${time}] "<span class="record-item-player">${this.player.name}</span>"打败了"<span class="record-item-npc">${this.npc.name}</span>"！`);
        }
        if (this.npc.hp > 0) {
          this.output(`[${time}] "<span class="record-item-player">${this.player.name}</span>"被"<span class="record-item-npc">${this.npc.name}</span>"打败了。。。`);
        }
      },
      output(content) {
        this.records.push(content);
        this.$nextTick(_ => {
          const recordBox = this.$refs['record_box'];
          recordBox.scrollTop = recordBox.scrollHeight;
        });
      }
    },
    created() {
      this.chooseSkill(this.player);
      this.chooseSkill(this.npc);
      this.timer();
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .container {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
    width: 100%;
  }

  .process-box {
    position: relative;
    margin: 0 60px;
    width: 10px;
    height: 300px;
    border-radius: 10px;
    background-color: cornflowerblue;
  }

  .process-item {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 40px;
    height: 40px;
    border-radius: 40px;
  }

  .process-item:after {
    content: '';
    position: absolute;
    right: -3px;
    top: 15px;
    width: 10px;
    height: 10px;
    background-color: lightblue;
    transform: rotate(45deg);
  }

  .process-player {
    left: -50px;
    background-color: lightblue;
  }

  .process-player:after {
    right: -3px;
  }

  .process-npc {
    right: -50px;
    background-color: lightblue;
  }

  .process-npc:after {
    left: -3px;
  }

  .text-box {
    flex: 1;
    display: flex;
    flex-direction: column;
    height: 300px;
  }

  .info-box {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
  }

  .info-item {
    flex: 1;
  }

  .info-hp {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 50%;
    height: 15px;
    border-radius: 2px;
    background-color: rgba(74, 74, 74, 0.2);
    border: 1px solid darkgray;
  }

  .info-hp-inner {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    border-radius: 0 4px 4px 0;
    background-color: yellowgreen;
    transition: all 1s;
  }

  .info-hp-num {
    color: #fff;
    font-size: 12px;
    z-index: 10;
  }

  .record-box {
    flex: 1;
    background-color: aliceblue;
    overflow: auto;
  }

  .record-list {
    list-style-type: none;
  }

  .record-item {
    text-align: left;
  }

  .record-item >>> .record-item-player {
    color: dodgerblue;
  }

  .record-item >>> .record-item-npc {
    color: coral;
  }

  .record-item >>> .record-item-damage {
    color: brown;
  }

  .record-item >>> .record-item-skill {
    color: cornflowerblue;
  }

  .record-item-heal {
    color: chartreuse;
  }
</style>
