---
title: Combat Math
nav_order: 103
---

# Combat Math

## The mechanics of combat

- The attacker rolls their weapon die and subtracts the target’s armor, then deals the remaining total to their opponent’s HP. Unarmed attacks always do 1d4 damage.
- If multiple attackers target the same foe, roll all damage dice and keep the single highest result.
- Damage that reduces a target’s HP below zero decreases a target’s STR by the amount remaining. They must then make a STR save to avoid critical damage. 
    - Any PC that suffers critical damage cannot do anything but crawl weakly, grasping for life.  If given aid and rest, they will stabilize. If left untreated, they die within the hour.
- If a PC’s STR is reduced to 0, they die.

## Typical PC
- 3d6 for STR (Let's say 11)
- 1d6 starting HP (Let's call it 4)
- Say 2 armor as baseline. (Chainmail or Brigadine+helmet, etc.)
- Let's give em a sword for d8 damage


## Example Monster

Grizzly Bear
: 6 HP, 15 STR, claws (d8)

Root Goblin
: 4 HP, 8 STR, 14 DEX, 8 WIL, spear (d6)

Hooded Men
: 12 HP, 9 STR, 12 DEX, 14 WIL, leystaff (d8), 


## Combat Simulation

- The Pc and the monster just start wailing on each other, attacking back and forth.
- No impaired attacks or special moves or spells or anything clever.
- All critical injuries are treated as death
- 1,000,000 simulated combat instances

### Bear vs Standard Reference Adventurerer

#### Adventurer goes first
- Adventurer win rate: 0.64
- Average number of rounds: 2.65
- When Adventurer survives, they're left with an average of 1.81 HP and  10.44 STR
- When Bear survives, they're left with an average of 0.93 HP and  12.80 STR

#### Bear goes first
- Bear win rate: 0.57
- Average number of rounds: 2.71
- When Bear survives, they're left with an average of 2.36 HP and  13.84 STR
- When Adventurer survives, they're left with an average of 1.08 HP and  9.95 STR


### Root Goblin vs Standard Reference Adventurerer

#### Adventurer goes first
- Adventurer win rate: 0.96
- Average number of rounds: 1.86
- When Adventurer survives, they're left with an average of 2.81 HP and  10.95 STR
- When Goblin survives, they're left with an average of 0.28 HP and  6.26 STR

#### Goblin goes first
- Goblin win rate: 0.14
- Average number of rounds: 2.68
- When Goblin survives, they're left with an average of 0.89 HP and  7.12 STR
- When Adventurer survives, they're left with an average of 1.75 HP and  10.78 STR


### Hooded Man vs Standard Reference Adventurerer

#### Adventurer goes first
- Adventurer win rate: 0.46
- Average number of rounds: 3.20
- When Adventurer survives, they're left with an average of 1.13 HP and  10.02 STR
- When Hooded Man survives, they're left with an average of 3.93 HP and  8.67 STR


#### Hooded Man goes first
- Hooded Man win rate: 0.73
- Average number of rounds: 2.93
- When Hooded Man survives, they're left with an average of 6.24 HP and  8.83 STR
- When Adventurer survives, they're left with an average of 0.72 HP and  9.45 STR


<details closed markdown="block">
<summary>Psst, you wanna see some janky code?</summary>
```python
import random
import math



#%% Create some dudes

class Actor(object):
    Name = None
    HP = 0
    STR = 0
    Armor = 0
    Attack = 0
    Alive = True

    # The class "constructor" - It's actually an initializer 
    def __init__(self, Name, HP,STR,Armor,Attack):
        self.Name = Name
        self.HP = HP
        self.STR = STR
        self.Armor = Armor
        self.Attack = Attack

def make_actor(Name,HP,STR,Armor,Attack):
    actor = Actor(Name,HP,STR,Armor,Attack)
    return actor

#%%

adventurerStats = ["Adventurer",4,12,2,8]    
bearStats = ["Bear",6,15,0,8]  
goblinStats = ["Goblin",4,8,0,6]
cultistStats = ["Hooded Man",12,9,0,8]

#%%

def runTrial(stats0,stats1):    
    actor0 = make_actor(*stats0)
    actor1 = make_actor(*stats1)
    actors = [actor0,actor1]
    
    
    turns = [([actor0.Name,actor0.HP,actor0.STR,actor0.Alive,],[actor1.Name,actor1.HP,actor1.STR,actor1.Alive,])]
    
    #startingActor = random.randrange(2)
    startingActor = 0 #deterministic
    activeActor = actors[startingActor]
    otherActor = actors[1-startingActor]
    
    #print("First attack by ",activeActor.Name)
    
    while actor0.Alive and actor1.Alive:
        damage = random.randrange(activeActor.Attack) + 1 - otherActor.Armor
        damage = max(0,damage) #Don't want a weak attack to heal the target
        otherActor.HP -= damage
        if otherActor.HP < 0:
            otherActor.STR += otherActor.HP
            otherActor.HP = 0
            if otherActor.STR <= 0:
                otherActor.Alive = False
            else:
                STRsave = random.randrange(20)+1
                if STRsave > otherActor.STR:
                    otherActor.Alive = False
                      
        turns.append(([actor0.Name,actor0.HP,actor0.STR,actor0.Alive,],[actor1.Name,actor1.HP,actor1.STR,actor1.Alive,]))
        
        #switcheroo
        _ = activeActor
        activeActor = otherActor
        otherActor = _
    
    return actor0,actor1, turns
        
#results = runTrial(adventurerStats,bearStats,)
    
#%% Run 1000 trials and print results
    
numTrials = 1000000
stats0 = cultistStats
stats1 = adventurerStats

trialRecords = []

for _ in range(numTrials):
    results = runTrial(stats0,stats1,)
    trialRecords.append(results)

agent0WinPercent = sum([record[0].Alive for record in trialRecords])/numTrials
print(stats0[0], "win rate:", "%.2f" % agent0WinPercent)

avgNumberRounds = sum([math.ceil(len(record[2])/2) for record in trialRecords])/numTrials
print("Average number of rounds:", "%.2f" % avgNumberRounds)

avgRemainingHP0 = 0
avgRemainingSTR0 = 0
avgRemainingHP1 = 0
avgRemainingSTR1 = 0
for record in trialRecords:
    if record[0].Alive:
        avgRemainingHP0 += record[0].HP
        avgRemainingSTR0 += record[0].STR
    elif record[1].Alive:
        avgRemainingHP1 += record[1].HP
        avgRemainingSTR1 += record[1].STR
        
avgRemainingHP0 = avgRemainingHP0 / sum([record[0].Alive for record in trialRecords])
avgRemainingSTR0 = avgRemainingSTR0 / sum([record[0].Alive for record in trialRecords])
avgRemainingHP1 = avgRemainingHP1 / sum([record[1].Alive for record in trialRecords])
avgRemainingSTR1 = avgRemainingSTR1 / sum([record[1].Alive for record in trialRecords])
    
print("When",stats0[0],"survives, they're left with an average of", "%.2f" % avgRemainingHP0,"HP and ","%.2f" % avgRemainingSTR0,"STR")
print("When",stats1[0],"survives, they're left with an average of","%.2f" % avgRemainingHP1,"HP and ","%.2f" % avgRemainingSTR1,"STR")
        
```
</details>

