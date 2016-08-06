---
title: Types of Decomposition
---

# No Decomposition

damage = attack_rating * Random.new.rand(0.9..1.0) * (256.0 - defense_rating) / 256.0 + 1

# Reassignment

The steps are different values on the same variable.

damage = attack_rating
damage *= Random.new.rand(0.9..1.0)
damage *= (256.0 - defense_rating) / 256.0
damage += 1

# Constant Assignment

The steps are different variables.

attack_modifier = Random.new.rand(0.9..1.0)
attack_damage = attack_rating * attack_modifier
defense_modifier = (256.0 - defense_rating) / 256.0
damage_after_defense = attack_damage * defense_modifier
damage_with_scratch = damage_after_defense + 1

# Method Chaining

The steps are different modified versions of the same object. (Or different objects.)

damage = Damage.new(attack_rating).randomize.apply_defense(defense_rating).apply_scratch_damage

# Functional Chaining

The steps are return values of different functions.

apply_scratch_damage(apply_defense(defense_rating, randomize_attack(attack_rating)))

def randomize_attack(attack_rating)
  attack_rating * Random.new.rand(0.9..1.0)
end

def apply_defense(damage, defense_rating)
  damage * damage_modifier(defense_rating)
end

def damage_modifier(defense_rating)
  (256.0 - defense_rating) / 256.0
end

def apply_scratch_damage(damage)
  damage + 1
end

# Private Method Tree

The steps are different methods that call each other.

```
def damage
      (attack_damage * defense_modifier) + scratch_damage
    end

    def attack_damage
      attack_rating * attack_modifier
    end

    def attack_modifier
      Random.new.rand(0.9..1.0)
    end

    def defense_modifier
      (max_defense_rating - defense_rating) /
        max_defense_rating
    end

    def max_defense_rating
      MAX_DEFENSE_RATING
    end
    
    def scratch_damage
      1
    end
```
