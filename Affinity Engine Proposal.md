# Terrify Affinity Engine

## NOTE: This document contains technical

## implementation details - Enough for a

## competitor to produce the Affinity Engine. Do

## not share without redacting/removing key

## information

## Executive Summary

The Terrify Affinity Engine is a powerful, graph-based matchmaking system
that connects content producers with relevant brand partners through an
intelligent data processing pipeline. By analyzing relationships between
entities (people, brands, places, emotions, etc.) as semantic triples, the
engine identifies high-value partnership opportunities that might otherwise
remain undiscovered.

The Affinity Engine revolutionizes how media professionals and brands
connect by eliminating traditional networking barriers, leveraging real-time
data collection, and providing actionable partnership recommendations
backed by data-driven insights.

While our initial focus is on media partnerships, the core technology is
fundamentally universal. At its essence, the Affinity Engine answers two
profoundly simple yet powerful questions: “Who are you?” and “What do you
want?” This elegant simplicity makes the technology applicable across
countless domains. The same underlying engine could help people find the
perfect restaurant based on their taste profile, plan an ideal vacation that
matches their interests and constraints, discover their next favorite book, or
even find compatible dating partners. Any scenario where matching
preferences to possibilities is valuable represents a potential application of
the Affinity Engine’s core technology. This universality creates enormous
scaling potential beyond our initial market focus.

## Historical Context & Opportunity

### The Evolution of Knowledge Graphs

Historically, building comprehensive knowledge graphs has been
prohibitively difficult for most businesses:

```
Pre-2010 Era : Knowledge extraction required manual curation or
brittle rule-based systems
2010-2020 Era : Companies like Google and Facebook built proprietary
knowledge graphs through:
Massive web scraping operations
```

```
Wikipedia/Wikidata extraction
Expensive human verification processes
```
These approaches required enormous resources, making similar capabilities
inaccessible to most businesses.

### The AI-Driven Revolution

The landscape has fundamentally changed with recent advancements:

```
Transformer Models : Modern large language models can extract
semantic relationships with unprecedented accuracy
Real-Time Integration : Services like Grok with direct Twitter
integration enable immediate access to current information
Reduced Infrastructure Requirements : Cloud-based AI processing
eliminates need for massive on-premises computing
```
This convergence creates a unique **patentable opportunity** to build an
accessible, real-time knowledge graph system before competitors recognize
the potential and attempt to replicate it.

### The Patent Opportunity

The Terrify Affinity Engine represents a novel combination of: - Real-time
triple extraction methodology - Specialized scoring algorithms for
partnership valuation - Unique graph traversal approaches for affinity
discovery

As history demonstrates, once a working system is demonstrated,
competitors will inevitably attempt to replicate it (“I didn’t think it was
possible! But you proved it was... so all I have to do NOW is learn how YOU
did it and copy you!”). Securing patents on key methodologies represents a
critical strategic advantage.

## “We Just Have Two Questions, And We Can

## Find Exactly What You Need”

The magic of the Affinity Engine lies in its radical simplicity. No complicated
forms. No lengthy onboarding process. Just two straightforward questions:

```
“What do you do?” (Who are you?)
“What do you WANT to do?” (What are you looking for?)
```
Users answer these questions in natural language—just as they would
explain to a colleague or friend. No need for technical jargon, specialized
formatting, or understanding of how the system works. They simply speak
their truth.

For example: * “I produce a weekly TV show about makeup artists
competing for a contract” * “I want to find brands that would sponsor our
next season”


**The Timeline Experience:** 1. User answers the first question (5-
seconds) 2. While answering the second question, we’re already processing
their first answer (10-15 seconds) 3. After completing both questions, results
appear within: * 20-30 seconds for first-time searches (when we need to
build the knowledge graph) * 1-2 seconds for subsequent searches
(leveraging existing graph data)

If users don’t find exactly what they’re looking for, refinement is instant.
They can simply tap either question to modify their response, triggering a
fresh search that takes only seconds since we’ve already ingested their base
triples. This allows for an iterative discovery process that feels
conversational rather than transactional.

The system gets smarter with every interaction—not just from this user, but
from all users—creating a virtuous cycle where search quality and speed
continuously improve. This elegant simplicity masks the sophisticated
technology working behind the scenes, making the experience feel like
magic to the end user.

## How It Works

### Core Concept

The Affinity Engine operates on a simple premise: everything and everyone
has connections that can be mapped and leveraged. These connections are
stored as “triples” in the format:

[Subject, Predicate, Object]

For example:

["American Beauty Star", "hosted by", "Ashley Graham"]

["Revlon", "sponsored", "American Beauty Star Season 2"]

["Millennials", "engage with", "beauty content on social media"]

When combined at scale, these triples form a comprehensive knowledge
graph that reveals direct and indirect relationships between entities,
enabling the identification of synergistic partnership opportunities.

### Systems Theory & Emergent Knowledge

The Affinity Engine leverages principles from systems theory to create a
solution greater than the sum of its parts:

```
Emergent Properties
Individual triples represent atomic facts
At scale, these interconnected triples create emergent knowledge
structures
The more triples exist about an entity (e.g., Revlon), the more
detailed the systemic understanding becomes
```

```
This emergent complexity cannot be predicted by analyzing
individual triples in isolation
Knowledge Substrate Formation
As triple density increases, a “knowledge substrate” forms
This substrate enables:
Inference of unstated relationships
Identification of patterns across seemingly unrelated domains
Prediction of future relationships based on structural
similarities
Generation of new knowledge rather than merely organizing
existing information
Synergistic Combination with RAG
The knowledge graph serves as a specialized index for Retrieval-
Augmented Generation
RAG processes can target specific relationship paths identified in
the graph
The combination enables precision targeting of relevant
information while maintaining natural language flexibility
Adaptive System Properties
The system becomes increasingly intelligent as it processes more
data
Self-correction mechanisms ensure consistency across the
knowledge graph
Redundant pathways create robustness against incomplete or
incorrect data
```
The graph database doesn’t just store information—it creates a foundation
for computational knowledge discovery that was previously impossible
without massive resource investment.

### User Flow

```
Simplified Onboarding (< 60 seconds)
Answer two questions:
Q1: “What do you do?” (e.g., “I produce American Beauty
Star, a weekly TV show on Lifetime”)
Q2: “What do you WANT to do?” (e.g., “I want to find
marketing deals”)
While answering, background processes capture and validate
relevant information
No tedious verification steps - first interactions focus on value
demonstration
Real-time Graph Generation
System automatically builds entity relationships by gathering data
from:
Social media platforms (Twitter, Facebook, etc.)
Industry databases (IMDB, etc.)
News sources
Web content
Information is structured into semantic triples across categories:
Facts
People
```


```
Places
Things
Brands
Emotions
Colors
Genres
Intelligent Match Generation
Proprietary algorithms identify high-potential partnerships
Matches are scored based on:
Connection strength (direct vs. indirect relationships)
Brand budget allocation patterns
Historical partnership success rates
Audience demographic overlap
Cultural and genre alignment
Intuitive Deal Discovery Interface
Users are presented with potential matches in a familiar card-
swiping interface
Swipe right to save deals of interest for further review
Swipe left to dismiss deals that aren’t relevant
Each card displays key match information and estimated value
This “Tinder for deals” approach creates a low-friction experience
that feels natural to users
Saved deals are stored in a personalized dashboard for more
detailed review
Actionable Recommendations
Users receive ranked partnership opportunities
Each recommendation includes:
Brand/partner contact information
Draft partnership proposal template
AI-generated structure suggestions
Creative concept ideas
Value estimation based on comparable deals
```
## Technical Architecture

### AI Model Fine-Tuning Strategy

While the current implementation leverages off-the-shelf models such as
Claude 3.7 Sonnet to extract semantic triples and generate
recommendations, the Affinity Engine’s architecture includes a clear path to
performance optimization:

```
Initial MVP Approach
Utilizing Claude 3.7 Sonnet for semantic triple extraction and
generation
This approach enables rapid development and validation of core
concepts
Current performance is already sufficient for demonstrating
business value
```

```
Fine-Tuning Roadmap
As usage scales, fine-tuning a dedicated model like DeepSeek R
becomes economically viable
Targeted fine-tuning focused on:
Optimizing triple extraction precision for media and brand
entities
Reducing latency for real-time processing
Improving specificity of relationship identification
This fine-tuning requires a relatively modest budget compared to
from-scratch model development
Expected outcome: 3-5x improvement in processing time with
higher accuracy
Specialized RAG Implementation
Custom RAG (Retrieval Augmented Generation) framework
optimized for graph database queries
Specialized embedding approach for triple relationships
Integration of temporal relevance scoring for recency-sensitive
queries
```
This thoughtful approach to AI implementation balances the need for rapid
market entry with a clear path to technical optimization, ensuring the
Affinity Engine remains both cutting-edge and economically viable as it
scales.

### Data Collection Layer

```
Real-time Web Scraping Engine
Intelligent crawling of relevant sources based on user inputs
Natural language processing to extract semantic relationships
Entity recognition and categorization
API Integration Hub
Connections to leading social platforms and content databases
Rate-limiting and quota management
Data normalization and standardization
```
### Knowledge Processing Core

```
Triple Extraction System
Advanced NLP models identify subject-predicate-object
relationships
Entity disambiguation and resolution
Confidence scoring for extracted information
Graph Database
High-performance triple store optimized for relationship queries
Temporal awareness (data freshness tracking)
Efficient indexing for rapid retrieval
```
### Match Generation Engine

```
Multi-dimensional Scoring Algorithm
Weighted evaluation of direct and indirect connections
```

```
Consideration of business factors (budgets, previous deal
structures)
Audience affinity measurement
Recommendation Synthesis
AI-powered deal structure suggestion
Value proposition generation
Personalized opportunity ranking
```
### User Interface

```
Progressive Disclosure Flow
Minimal initial information requirements
Real-time feedback during processing
Clear visualization of connection pathways
Deal Workspace
Deal template generator
Negotiation tracking
Success measurement tools
```
## Key Advantages

### For Content Producers

```
Discover untapped partnership opportunities beyond obvious
direct connections
Save hundreds of hours normally spent on research and outreach
Data-backed negotiation leverage with comprehensive relationship
insights
Rapid deal generation with proposal templates and contact
information
```
### For Brands

```
Find authentic content partnerships aligned with brand values and
audience
Discover emerging properties before they become prohibitively
expensive
Optimize marketing spend with tailored partnership
recommendations
Access comprehensive audience insights for potential partners
```
### For Both Sides

```
Reduce partnership friction with clear value propositions
Minimize research overhead through automated data gathering
Increase deal success rates with better-matched partnerships
Discover non-obvious connections that create unique value
```
#### ◦ ◦ • ◦ ◦ ◦ • ◦ ◦ ◦ • ◦ ◦ ◦ • • • • • • • • • • • •


## Technical Advantages

```
Self-improving system - Knowledge graph becomes more valuable
with each query
Real-time data collection - No reliance on potentially outdated
databases
Scalable architecture - Handles entities from niche to mainstream
Minimal storage requirements - Triplet data is compact and
efficiently indexed
Rapid results - Leverages existing knowledge graph for instant
matching when available
```
## Potential Challenges & Solutions

### 1. Data Freshness

**Challenge** : The quality of recommendations depends on how current the
triple data is, especially for fast-moving industries.

**Solution** : Our system automatically refreshes triples older than 30-90 days.
When users query the system, any data points that exceed our freshness
threshold are updated in real-time, ensuring recommendations always
reflect current reality.

### 2. Cold Start Problem

**Challenge** : New systems typically struggle with limited initial data.

**Solution** : There is effectively no cold start problem with our approach. If
triples don’t exist in our database, we collect them in real-time during the
search process. While this might extend processing time by 15-20 seconds
initially, the system delivers value from day one and continuously improves
as the knowledge graph expands.

### 3. Accuracy of Auto-filled Information

**Challenge** : Allowing users to “pretend” during onboarding could lead to
quality issues.

**Solution** : The system is designed to be impervious to incorrect self-
reporting. Since we build triples based on real-world web data, not user
claims, the system will always generate accurate relationship data
regardless of what users say about themselves. If a user claims to be
someone they’re not, the system will simply gather real triples for that
entity. These triples are automatically purged after 90 days if unused,
representing a negligible storage cost (less than $0.001).


### 4. Privacy Considerations

**Challenge** : Gathering and connecting data from multiple sources raises
privacy concerns.

**Solution** : The Affinity Engine exclusively uses properly licensed API access
to all data sources, ensuring full compliance with terms of service. We
process only publicly available information and maintain strict data
governance practices that align with global privacy regulations.

## Business Model

### Freemium Approach

```
Free Tier
Limited number of searches per month
Basic match recommendations
Standard templates
Professional Tier
Unlimited searches
Advanced matching algorithms
Premium templates and proposal tools
Deal tracking and analytics
Enterprise Tier
Custom integration options
Dedicated support
White-label solutions
Advanced analytics and reporting
```
### Value-Based Pricing

The platform can potentially capture a small percentage of successful deals
facilitated through the system, creating perfect alignment between Terrify’s
success and customer outcomes.

## Competitive Advantage

While other platforms attempt to connect brands and content creators:

```
Traditional agencies rely on personal networks and manual research
Influencer platforms focus on social metrics rather than deep
relationship mapping
Marketing databases offer static information without intelligent
matching
```
The Affinity Engine combines the best elements of all three approaches
while eliminating their limitations through:

```
Speed - Results in seconds, not days or weeks
```


```
Depth - Multi-dimensional relationship mapping beyond surface-level
connections
Intelligence - AI-powered matching and recommendation generation
Actionability - Complete workflow from discovery to proposal
```
## Implementation Timeline

```
Phase 1: Core Engine Development (3 months)
Triple extraction system
Basic graph database
Simple user interface
Phase 2: Match Generation Refinement (2 months)
Scoring algorithm development
Recommendation engine
Enhanced user experience
Phase 3: Full Feature Set (3 months)
Deal workspace
Templates and proposal tools
Analytics dashboard
Phase 4: Scale and Optimize (Ongoing)
Performance optimization
Additional data source integration
Advanced match algorithms
```
## Technical Implementation Details

### Triple Data Structure Examples

The following examples demonstrate the actual data structures used by the
Affinity Engine, providing insight into the system’s operation at the data
level.

**Fact Triples Example**


```
["American Beauty Star", "has typical audience", "individuals
interested in beauty industry"],
["typical audience", "includes", "younger viewers seeking hair
and makeup tips"],
["typical audience", "enjoys", "reality competition shows"],
["American Beauty Star", "appeals to", "fans of beauty
transformations"],
["typical audience", "consists of", "aspiring makeup artists"],
["typical audience", "includes", "hair styling enthusiasts"],
["typical audience", "watches for", "creative beauty
techniques"],
["American Beauty Star", "targets", "people passionate about
cosmetics"],
```

**Person Triples Example**

**Places Triples Example**

```
["typical audience", "enjoys", "drama of competition"],
["typical audience", "seeks", "inspiration for personal beauty
routines"]
```
]



```
["Jessica Belkin", "competed in", "American Beauty Star Season
1"],
["Jessica Belkin", "was", "a contestant"],
["Andrew Velazquez", "competed in", "American Beauty Star
Season 1"],
["Andrew Velazquez", "opened", "his own salon"],
["Amanda Terry", "competed in", "American Beauty Star Season
1"],
["Amanda Terry", "works as", "makeup artist"],
["Catherine Shim", "competed in", "American Beauty Star Season
1"],
["Catherine Shim", "runs", "Instagram with over 1M followers"],
["Corey Ford", "competed in", "American Beauty Star Season 1"],
["Corey Ford", "is", "hair stylist from Houston"]
```
]



```
["Los Angeles, California", "hosted", "American Beauty Star
Season 1 filming"],
["Los Angeles, California", "is home to", "Lifetime network
studios"],
["Atlanta, Georgia", "hosted",
"American Beauty Star Season 2 filming"],
["Atlanta, Georgia", "provided", "new production setting for
Season 2"],
["Universal CityWalk, Los Angeles", "featured in", "Season 1
makeover challenge"],
["Universal CityWalk, Los Angeles", "is", "popular public
shopping area"],
["New York City, New York", "linked to", "Teen Vogue prize
editorial shoot"],
["New York City, New York", "houses", "Hearst Magazines
headquarters"]
```
]


**Things Triples Example**

**Affinity Demographic Matches Example**

**Deal Connection Example**



```
["Revlon ColorStay Makeup", "sponsored", "American Beauty
Star Season 2"],
["Revlon ColorStay Makeup", "used for", "foundation in
challenges"],
["Revlon PhotoReady Concealer", "featured in", "American
Beauty Star Season 2"],
["Revlon PhotoReady Concealer", "applied in", "contestant
makeovers"],
["Revlon Super Lustrous Lipstick", "provided for", "American
Beauty Star contestants"],
["Revlon Super Lustrous Lipstick", "enhanced", "red carpet
looks"],
["Revlon Professional Uniq One", "used in", "American Beauty
Star hair styling"],
["Revlon Professional Uniq One", "won as", "part of Season 2
prize package"]
```
]



```
["Women aged 18-34", "are typical audience for", "American
Beauty Star"],
["Women aged 18-34", "have interest level", "high"],
["Beauty enthusiasts (all genders, 18-44)", "are typical
audience for", "American Beauty Star"],
["Beauty enthusiasts (all genders, 18-44)", "have interest
level", "high"],
["Gen Z females (18-24)", "are typical audience for",
"American Beauty Star"],
["Gen Z females (18-24)", "have interest level", "high"],
["Millennial women (25-34)", "are typical audience for",
"American Beauty Star"],
["Millennial women (25-34)", "have interest level", "high"]
```
]


```
"id": "1a1b1c1d-0001-4a4a-aaaa-111111111111",
"subject": "Revlon",
"subject_type": "brand",
```

**Brand Recommendations Example**

```
"predicate": "associated_with",
"object": "Ashley Graham",
"object_type": "person"
},
{
"id": "1a1b1c1d-0002-4a4a-aaaa-111111111111",
"subject": "Revlon",
"subject_type": "brand",
"predicate": "associated_with",
"object": "beauty",
"object_type": "genre"
},
{
"id": "1a1b1c1d-0003-4a4a-aaaa-111111111111",
"subject": "Ashley Graham",
"subject_type": "person",
"predicate": "appears_in",
"object": "American Beauty Star",
"object_type": "show"
},
{
"id": "1a1b1c1d-0004-4a4a-aaaa-111111111111",
"subject": "American Beauty Star",
"subject_type": "show",
"predicate": "aired_on",
"object": "Lifetime",
"object_type": "network"
},
{
"id": "1a1b1c1d-0005-4a4a-aaaa-111111111111",
"subject": "Revlon",
"subject_type": "brand",
"predicate": "targets_audience",
"object": "fashion-forward women 18-34",
"object_type": "audience_segment"
}
```
]


```
"brand": "Revlon",
"score": 95 ,
```

```
"match_reasons": [
"Direct association with Ashley Graham (show host & brand
ambassador)",
"Shared genre: beauty",
"Shared audience: fashion-conscious women 18-34"
],
"potential_value_factors": [
"Revlon has a large ad budget",
"High product-audience alignment",
"Boosts credibility by reinforcing celebrity endorsement"
],
"notes": "This is the textbook definition of a synergistic
brand-content match."
```
},

{

```
"brand": "Glossier",
"score": 83 ,
"match_reasons": [
"Shared genre: beauty",
"Millennial/Gen Z female target demo overlaps with show
audience"
],
"potential_value_factors": [
"Strong digital engagement",
"Brand built on influencer marketing, perfectly aligned
with show format"
],
"notes": "Could benefit from the show's glam context to
elevate premium positioning."
```
},


```
"brand": "L'Oréal",
"score": 78 ,
"match_reasons": [
"Shared genre: beauty",
"Presence in youth, fashion, and cosmetic markets"
],
"potential_value_factors": [
"Global reach with local ad spend flexibility",
"Can use tie-in for campaign around diversity and
inclusion"
],
"notes": "Corporate powerhouse that could tie this into a
broader initiative."
```

### Technical Advantages for System Architects

From a systems architecture perspective, the Terrify Affinity Engine offers
several technical advantages:

```
1. Horizontally Scalable : The triple-based data model scales linearly
with data volume
2. Eventual Consistency Model : Allows for rapid updates without
blocking queries
3. Fault Tolerance : Redundant relationship paths create system
resilience
4. Low Latency Design : Graph traversal optimized for relationship
discovery
5. Efficient Storage : Triple format minimizes storage requirements
compared to traditional relational databases
6. Federated Processing : Query execution distributed across processing
nodes
7. Schema Flexibility : No rigid schema requirements enables rapid
iteration and extension
```
## Conclusion

The Terrify Affinity Engine represents a paradigm shift in how media
professionals and brands discover and establish partnerships. By harnessing
the power of relationship data and intelligent matching, we eliminate the
inefficiencies of traditional networking while creating opportunities for more
authentic, valuable collaborations.

Our approach delivers immediate value to users from day one while
continuously improving as more entities and relationships are added to the
knowledge graph. The system’s ability to identify non-obvious connections
creates unique partnership opportunities that would likely remain
undiscovered through conventional methods.

The implementation leverages cutting-edge AI capabilities that have only
recently become viable, creating a window of opportunity for intellectual
property protection before competitors fully grasp the methodology’s
potential. The knowledge substrate that emerges from our triple-based
approach generates insights that transcend the individual data points,
creating a competitive advantage that grows stronger over time.

With a minimal onboarding process, real-time data collection, and actionable
recommendations, the Affinity Engine is positioned to become the essential
tool for anyone seeking to build strategic partnerships in the content and
brand space.



