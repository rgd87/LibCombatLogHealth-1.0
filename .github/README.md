# LibCombatLogHealth-1.0

## **OBSOLETE IN SHADOWLANDS**

Combat log events occur a lot more frequentrly than UNIT_HEALTH,
this library tracks incoming healing and damage and adjusts health values.
As a result you can see health updates sooner and more often.

This implementation is safe and accurate.
For each unit we keep history of health values after each change from combat log.
When UNIT_HEALTH arrives, UnitHealth value is searched in this log.
If it's found, then chain is valid, and library proceeds to return latest value from it.
If not it falls back onto UnitHealth value. If UnitHealth values in the next 1.4 seconds
also could not be found to re-validate combat log chain,
then it is reset with current UnitHealth value as a starting point.

Why it's like that and not simplier
-----------------------------------

UNIT_HEALTH and CLEU are asynchronous, UNIT_HEALTH throttles and is usually slower,
but sometimes it comes first. And with CLEU immediately after it double damage/healing occurs.
I'm avoiding that, keeping them separate and only checking whether
combat log value has deviated from UnitHealth.

UNIT_HEALTH_FREQUENT
--------------------

This new event, that was introduced in 5.0, still throttles damage, but not heals.
Or at least it doesn't mash up heals with damage.
In short, if you just listen to both UNIT_HEALTH and UNIT_HEALTH_FREQUENT,
that's a decent compromise.

Shadowlands
--------------------

In 9.0 there's a new UNIT_HEALTH that is very fast. By the look of it, it's as fast as combat log.

Usage:
---------
```lua
local f = CreateFrame("Frame") -- your addon
local LibCLHealth = LibStub("LibCombatLogHealth-1.0")
if LibCLHealth then
    f:UnregisterEvent("UNIT_HEALTH")
    LibCLHealth.RegisterCallback(f, "COMBAT_LOG_HEALTH", function(event, unit, eventType)
        -- eventType - either nil when event comes from combat log, or "UNIT_HEALTH" to indicate events that can carry  update to death/ghost states
        local health = LibCLHealth.UnitHealth(unit)
        print(event, unit, health)
    end)
end
```
