
# Experiment No. 9 — Building a Simple AI Agent: AI Tourist Guide for India

## Aim

To design, implement, and test a simple goal-based AI agent in Python that plans a personalized India trip itinerary based on a tourist's interests, trip duration, and daily budget.

## Objectives

- Understand the structure and working principles of a goal-based AI agent.
- Apply the PEAS framework to a travel-planning problem.
- Create a knowledge base of Indian tourist destinations.
- Implement perception, reasoning, planning, and action functions.
- Generate and test personalized, budget-aware travel itineraries.

## Introduction

An AI agent perceives its environment through sensors and acts upon it through actuators to achieve a specific goal. The PEAS framework describes an agent in terms of its **Performance measure, Environment, Actuators, and Sensors**.

Agents can be classified as simple reflex agents, goal-based agents, and utility-based agents. A goal-based agent selects actions that help achieve a specified goal.

In this experiment, an AI Tourist Agent for India is developed to recommend a trip itinerary based on a tourist's interests, available days, and daily budget. The agent uses a predefined destination knowledge base to filter suitable destinations, plan a day-wise itinerary, and present an estimated trip cost.

![AI travel planning concept](https://images.unsplash.com/photo-1488646953014-85cb44e25828?auto=format&fit=crop&w=1200&q=80)

## PEAS Framework

| Component | Description |
|---|---|
| **Performance Measure** | Interest matching, budget compliance, itinerary completeness, and useful recommendations |
| **Environment** | Indian tourist destinations and the tourist's preferences |
| **Actuators** | Displaying recommended destinations, itinerary details, and estimated costs |
| **Sensors** | User-provided interest category, trip duration, and daily budget |

## Tools and Technologies

- **Programming language:** Python 3
- **Library:** `textwrap`
- **Environment:** Python IDE or terminal

## System Architecture

```text
        ┌──────────────────────────┐
        │      Tourist Input       │
        │ Interest, Days, Budget   │
        └─────────────┬────────────┘
                      │
        ┌─────────────▼────────────┐
        │       PERCEIVE          │
        │ Read Tourist Profile    │
        └─────────────┬────────────┘
                      │
        ┌─────────────▼────────────┐
        │        REASON           │
        │ Filter Destinations by  │
        │ Interest and Budget     │
        └─────────────┬────────────┘
                      │
        ┌─────────────▼────────────┐
        │         PLAN            │
        │ Build Day-wise Itinerary│
        └─────────────┬────────────┘
                      │
        ┌─────────────▼────────────┐
        │          ACT            │
        │ Display Trip Plan       │
        │ and Estimated Cost      │
        └─────────────────────────┘
```

## Procedure

### Step 1: Import Required Libraries

The `textwrap` library is used to wrap long destination descriptions neatly when printing the itinerary.

```python
import textwrap
```

### Step 2: Define the Agent's Knowledge Base

The destination database is a list of dictionaries. Each destination contains its name, category, estimated daily cost, best travel season, and description.

```python
destinations = [
    {
        "name": "Taj Mahal, Agra",
        "category": "heritage",
        "cost_per_day": 3000,
        "best_season": "October to March",
        "description": "Visit the iconic marble mausoleum, a UNESCO World Heritage Site."
    },
    {
        "name": "Jaipur, Rajasthan",
        "category": "heritage",
        "cost_per_day": 2500,
        "best_season": "October to March",
        "description": "Explore royal palaces, forts, colourful markets, and historic architecture."
    },
    {
        "name": "Varanasi, Uttar Pradesh",
        "category": "spiritual",
        "cost_per_day": 2000,
        "best_season": "October to March",
        "description": "Experience the sacred ghats, temples, and spiritual traditions of the Ganges."
    },
    {
        "name": "Rishikesh, Uttarakhand",
        "category": "adventure",
        "cost_per_day": 3500,
        "best_season": "September to November and March to May",
        "description": "Enjoy river rafting, yoga, and outdoor adventures in the Himalayan foothills."
    },
    {
        "name": "Goa",
        "category": "beach",
        "cost_per_day": 4000,
        "best_season": "November to February",
        "description": "Relax on beaches, explore coastal towns, and enjoy local cuisine."
    },
    {
        "name": "Andaman Islands",
        "category": "beach",
        "cost_per_day": 5000,
        "best_season": "October to May",
        "description": "Discover clear waters, tropical beaches, and marine activities."
    },
    {
        "name": "Coorg, Karnataka",
        "category": "nature",
        "cost_per_day": 3000,
        "best_season": "October to March",
        "description": "Explore coffee plantations, green hills, waterfalls, and forest trails."
    },
    {
        "name": "Spiti Valley, Himachal Pradesh",
        "category": "adventure",
        "cost_per_day": 4500,
        "best_season": "May to October",
        "description": "Discover dramatic mountain landscapes, remote villages, and monasteries."
    },
    {
        "name": "Kaziranga, Assam",
        "category": "wildlife",
        "cost_per_day": 3500,
        "best_season": "November to April",
        "description": "Explore a renowned national park known for its one-horned rhinoceroses."
    },
    {
        "name": "Ooty, Tamil Nadu",
        "category": "hill_station",
        "cost_per_day": 2500,
        "best_season": "October to June",
        "description": "Enjoy cool weather, tea gardens, scenic viewpoints, and mountain railways."
    }
]
```

### Step 3: Perceive the Tourist's Preferences

The `perceive()` function represents the agent's sensors. It reads the tourist's interest, trip duration, and daily budget.

```python
def perceive(tourist):
    print("\n--- Tourist Profile ---")
    print(f"Interest: {tourist['interest']}")
    print(f"Trip Duration: {tourist['days']} days")
    print(f"Daily Budget: Rs. {tourist['budget']:,}")
    return tourist
```

### Step 4: Reason — Filter and Rank Destinations

The `reason()` function selects destinations matching the requested category and daily budget. If no destination fits the budget, it returns destinations in the requested category ordered from least to most expensive.

```python
def reason(tourist, destinations):
    interest = tourist["interest"].lower()
    budget = tourist["budget"]

    matching = [
        destination
        for destination in destinations
        if destination["category"] == interest
        and destination["cost_per_day"] <= budget
    ]

    if not matching:
        matching = [
            destination
            for destination in destinations
            if destination["category"] == interest
        ]

    return sorted(
        matching,
        key=lambda destination: destination["cost_per_day"]
    )
```

### Step 5: Plan — Build a Day-wise Itinerary

The `plan()` function allocates up to three days per destination. It moves to the next destination when the current allocation is complete and cycles through matching destinations if more days are required.

Consecutive days at the same destination are combined into one itinerary block. The total estimated cost is calculated from the daily cost and allocated days.

```python
def plan(tourist, matching_destinations):
    if not matching_destinations:
        return [], 0

    days_remaining = tourist["days"]
    itinerary = []
    destination_index = 0
    total_cost = 0

    while days_remaining > 0:
        destination = matching_destinations[
            destination_index % len(matching_destinations)
        ]

        allocated_days = min(3, days_remaining)

        itinerary.append({
            "destination": destination,
            "days": allocated_days
        })

        total_cost += destination["cost_per_day"] * allocated_days
        days_remaining -= allocated_days
        destination_index += 1

    return itinerary, total_cost
```

### Step 6: Act — Present the Itinerary

The `act()` function displays the itinerary, including destination details, travel season, allocated days, daily cost, and total estimated cost.

```python
def act(itinerary, total_cost):
    if not itinerary:
        print("\nNo destinations were found for this interest.")
        return

    print("\n" + "=" * 60)
    print("       PERSONALIZED INDIA TRAVEL ITINERARY")
    print("=" * 60)

    current_day = 1

    for item in itinerary:
        destination = item["destination"]
        days = item["days"]
        end_day = current_day + days - 1

        print(f"\nDays {current_day}-{end_day}: {destination['name']}")
        print(f"Category: {destination['category'].replace('_', ' ').title()}")
        print(f"Best Season: {destination['best_season']}")
        print(f"Daily Estimated Cost: Rs. {destination['cost_per_day']:,}")

        description = textwrap.fill(
            destination["description"],
            width=70,
            subsequent_indent="    "
        )
        print("Description:", description)

        current_day = end_day + 1

    print("\n" + "-" * 60)
    print(f"Total Estimated Trip Cost: Rs. {total_cost:,}")
    print("=" * 60)
```

### Step 7: Create the Agent Loop

The `run_agent()` function combines perception, reasoning, planning, and action into one complete agent cycle.

```python
def run_agent(tourist):
    tourist = perceive(tourist)
    matching_destinations = reason(tourist, destinations)
    itinerary, total_cost = plan(tourist, matching_destinations)
    act(itinerary, total_cost)
```

### Step 8: Test the Agent with Sample Tourist Profiles

Three tourist profiles are used to test the agent with different interests, trip durations, and budgets.

```python
tourists = [
    {
        "name": "Ananya",
        "interest": "heritage",
        "days": 6,
        "budget": 3500
    },
    {
        "name": "Rahul",
        "interest": "adventure",
        "days": 5,
        "budget": 5000
    },
    {
        "name": "Meera",
        "interest": "beach",
        "days": 7,
        "budget": 5500
    }
]

for tourist in tourists:
    print(f"\n\nTourist: {tourist['name']}")
    run_agent(tourist)
```

## Results and Observations

The agent was tested with three sample tourist profiles.

| Tourist | Interest | Duration | Daily Budget |
|---|---|---:|---:|
| Ananya | Heritage | 6 days | Rs. 3,500 |
| Rahul | Adventure | 5 days | Rs. 5,000 |
| Meera | Beach | 7 days | Rs. 5,500 |

The agent filters destinations by category and budget, generates a day-wise itinerary, and calculates the estimated trip cost.

### Sample Output

```text
Tourist: Ananya

--- Tourist Profile ---
Interest: heritage
Trip Duration: 6 days
Daily Budget: Rs. 3,500

============================================================
       PERSONALIZED INDIA TRAVEL ITINERARY
============================================================

Days 1-3: Jaipur, Rajasthan
Category: Heritage
Best Season: October to March
Daily Estimated Cost: Rs. 2,500
Description: Explore royal palaces, forts, colourful markets, and
    historic architecture.

Days 4-6: Taj Mahal, Agra
Category: Heritage
Best Season: October to March
Daily Estimated Cost: Rs. 3,000
Description: Visit the iconic marble mausoleum, a UNESCO World
    Heritage Site.

------------------------------------------------------------
Total Estimated Trip Cost: Rs. 16,500
============================================================
```

## Limitations

- Destination information and costs are predefined estimates.
- The agent does not retrieve live hotel, transport, or ticket prices.
- Travel time and distance between destinations are not considered.
- Seasonal recommendations are static and may not reflect current conditions.
- If no destination fits the budget, the agent may recommend destinations that exceed it.

## Conclusion

A simple goal-based AI Tourist Agent for India was successfully designed, implemented, and tested using Python. The agent follows the **Perceive → Reason → Plan → Act** cycle: it perceives a tourist's interests, duration, and budget; reasons over a destination knowledge base; plans a day-wise itinerary; and presents a trip recommendation with estimated costs.

This experiment demonstrates the fundamental components of autonomous agents, including environment knowledge, perception, reasoning, planning, and action. These concepts form the foundation for more advanced AI agents that use machine learning, real-time APIs, and large language models.
