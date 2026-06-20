# Chore Automations

## Scheduler

Automatically-resetting tasks for regular chores that use only the task's description for configuration!

Compatibility:
- The stock Local To-Do integration is what this was built for; mileage may vary
- `todo.update_item` must support: `due_date`, `rename`
- `todo.get_items` must return items specifying a `completed` date for items with `status = completed`

### Writing Recurring Tasks

The examples below show the title & description of to-do items that'll be managed by this automation.

Any items in your list without recurrence config are ignored!

---------------------------------

```md
# Clean The Cat's Litter Tray

Frequency: 2 days
```
_This task resets every other day - complete it any time on Monday, and it'll reset on Tuesday with a due date of Wednesday_

---------------------------------

```md
# Check the car's oil levels

Frequency: 1 month
Only: weekend
```
_Once per month, on Saturday or Sunday. If completed on a weekday, it will become due again the first Saturday after ~30 days have passed_

---------------------------------

```md
# Vacuum Upstairs

Frequency: 1 week
Only: weekend
Cycle:
- Vacuum Upstairs
- Vacuum Downstairs
```
_Once every 7 days, and only on Saturday or Sunday. Every time it resets, the title swaps between "Upstairs" and "Downstairs"_

---------------------------------

### Detailed Config

Most configuration text is case insensitive. Anything that _is_ case sensitive is called out as such!  
You can add these keywords anywhere in the task description.  
Generally, whitespace handling is flexible where it makes sense.

Tasks "reset" some time before they are next due. This is when the task is marked as "not complete" and the next due date is set.  
See [Idiosyncrasies](#Idiosyncrasies) for more detail!

#### Frequency

Sets how often the task will recur. If this isn't present, the automation does nothing with the task.  
The number can either be whole or fractional. The decimal point is not required for whole numbers.  
All units are converted to days; "1 month" is 30.4375 days.

```
Frequency: 0.0 [day|days|week|weeks|month|months|year|years]
```

Examples:
> `Frequency: 2 days`  
> `Frequency: 1.5  months`  
> `frequency:     1 week`  
> `FREQUENCY: 0.5 YEAR`

#### Only

Specifies a day-of-the-week constraint.  
Tasks that would be due on a day not satisfying the constraint will be scheduled on the next day that _does_.  
**Note: Not the closest day, a day _after_ the ideal due date.**

```
Only: [weekday|weekdays|weekend]
```

Examples:
> `Only: Weekend`  
> `Only: Weekdays`  
> `only:     weekend`  
> `ONLY: WEEKDAY`

#### Cycle

Specifies a repeating pattern of task titles.  
When a task resets, its title will be updated with the next title in the list, or loop back to the start.  

```
Cycle:
- Title 1
- Title 2
- Title 3
- etc.
```

Example:

```
Cycle:
- A
- B
- C
```

Important Notes:
- The keyword `Cycle:` must be at the start of a new line.
- No blank lines are supported - a blank line is considered the "end" of the cycle block.
- If the task's current title doesn't match any in the list, the first cycle title will be used.
- The logic uses an exact, **case-sensitive** match for the current title. If you have duplicate items in the cycle list, only the _last_ matching occurrence will ever match, so the cycle will be short-circuited.

Short-circuit example:

```
Cycle:
- A
- B
- C
- D
- E
- F
- C
- G
```

`C` is duplicated after `F`, resulting in:  
`A` → `B` → `C` → `G` → `A` → `B` → `C` (repeating)  

Manually setting the title to `D` will result in:  
`D` → `E` → `F` → `C` → `G` → `A` ... (you get the picture)

### Dogma

I find reoccurring calendar-driven tasks too rigid; If I miss a weekly chore by 3 days, I won't do it again 4 days later - it should be a full 7 days before it's due again!

Task reminders should tell me what to do, when to do it. Tasks that fit better on weekends should only show up on weekends.

Task setup / organisation should feel comfortable. I should be able to understand how often a task happens just by reading the task's description. I want home approval.

"Cycles" is the name of the concept of recurring tasks that also have a "sub-pattern". These should also fit into the solution neatly.

Examples of cycling tasks:
- "every 4th time I do laundry, I'll add a de-scaling tablet"
- "on week A I vacuum upstairs, on week B, downstairs"

The overarching goal is to make doing chores, and managing chores, as "human" as possible.
That's achieved by:
- Accepting that delays happen
- Reducing the "pressure" of "keeping up" with a rigid schedule
- Gentle reminders before something is due
- Setup/maintenance mustn't involve coding nor "excessively custom stuff"
- Hidden/transparent handling of constraints like "only on weekends"

As such this is an opinionated automation, but it'll probably integrate into your setup without conflicting with much.

**I welcome feedback and pull requests!**

### Idiosyncrasies

We think of recurring tasks in terms of frequency:
- Every 3 days  
- Once per week  
- 2 times per month  
- Quarterly

When a task is completed, it'll be due again one interval later:
> Weekly task, due 1st -  
> is completed on the 3rd.  
> Next due 10th, not 8th!

Some time before a task becomes due again, it is marked "incomplete" to serve as an "upcoming" reminder, and allow checking off early:
> | Task Frequency | it resets:       |
> |----------------|------------------|
> | ≤ 2 days       | one day before   |
> | ≤ 2 weeks      | 2.5 ≤ x ≤ 8 days |
> | ≤ 2 months     | 1 week before    |
> | ≤ 4 months     | 2 weeks before   |
> | ≤ 8 months     | 2 months before  |
> | ≤ 1.5 years    | 3 months before  |
> | and beyond...  | 4 months before  |

Recurring task config is written in a human-readable way in the task description, making it easy to edit & add new tasks without leaving the to-do list

One-time tasks (non-recurring) can live in the same list too 

Some tasks are best suited for weekdays, weekends, or specific days, so you can set constraints for that
