🍽️ Zomato AI Add-On Recommendation System
Smart Cart Intelligence for Average Order Value Optimization

👋 What Is This Project?
Ever noticed how the best restaurants always seem to know what you're going to want next — a Coke with your burger, a raita with your biryani? That's exactly what this project tries to teach Zomato to do, automatically and at scale.
This is a machine learning system that watches what a customer is building in their cart in real time and predicts the single most likely next item they'll want to add. It considers what's already in the cart, what time of day it is, which restaurant the order is from, and how that particular customer tends to order — then surfaces a timely, relevant add-on recommendation before they hit checkout.
Under the hood it's a LightGBM multi-class classifier trained on a sequential cart sampling framework, with a neat twist: we use sentence transformer embeddings to automatically categorise every menu item as a main, beverage, or dessert — no manual labelling required, which matters a lot when you're dealing with the wonderfully diverse landscape of Indian restaurant menus.

🚩 The Problem We're Solving
A large share of Zomato customers complete their orders without ever discovering complementary items that would genuinely improve their meal. The current "You May Also Like" surfaces rely on manually curated lists or global popularity rankings — they don't know that this particular customer, at 9 PM on a Friday, has already added a full main course and probably wants a dessert next, not another entrée.
The goal of this project is to fix that. Not with a generic recommendation, but with a contextually intelligent one that feels natural — the digital equivalent of a good waiter.

📦 The Dataset
We trained on a real-world Zomato order dataset from Delhi NCR: 21,342 orders placed across September 2024. Each record captures the full lifecycle of a delivered order — restaurant, items ordered, timestamps, discounts applied, delivery distance, and customer rating.
We filtered to only Delivered orders (roughly 18,900 records), then applied a sequential cart sampling transformation to generate approximately 15,000 supervised training samples from multi-item orders. The target label in each sample is the actual next item the customer added — making this a genuinely grounded, real-intent dataset rather than a synthetic one.

🛠️ How It Works
Step 1 — Data Preprocessing. We drop operational columns that carry no predictive signal (rider wait times, cancellation reasons, packaging charges), impute missing ratings with the column mean, normalise the distance field, and filter to delivered orders only. Timestamps are parsed into hour-of-day, day-of-week, and weekend flags.
Step 2 — Sequential Cart Sampling. This is where the magic happens. For every order with N items, we generate N–1 training samples. Sample 1 has one item in the cart and predicts the second; sample 2 has two items and predicts the third; and so on. This turns a recommendation problem into a clean, supervised classification task without needing any recurrent architecture.
Step 3 — Feature Engineering. We engineer ten features from the cart state and context: cart size, binary flags for whether the cart already has a beverage / dessert / main, order hour, meal period bucket, restaurant order volume, the customer's historical average items per order, and popularity and label-encoding of the last item added.
Step 4 — Semantic Item Categorisation. We load all-MiniLM-L6-v2 and embed every unique menu item. Each item is then assigned to whichever category — main, beverage, or dessert — its embedding sits closest to in cosine similarity space. This handles names like "Angara Grilled Paneer" or "Mango Mastani" gracefully, without a single hardcoded rule.
Step 5 — Model Training. LightGBM is trained on the 10 engineered features with an 80/20 train-test split. The output is a probability distribution over all ~500 unique items, from which we extract the top-K recommendations.

📊 Results
MetricValue🎯 Recall@10~82%📈 NDCG@10~0.71🔍 Precision@10~8.2%⚡ Inference latency<2ms per request🧪 Training samples~15,000🍱 Unique target items500+
To put those numbers in context: a global popularity baseline (always recommend the top-10 most ordered items) achieves Recall@10 of roughly 25%. Our model hits 82% — more than three times better — because it actually understands what's in the cart right now.

📁 Project Structure
├── NextBite.ipynb               # Main notebook: preprocessing → training → evaluation
├── Zomato_data.csv               # Raw order dataset (Delhi NCR, September 2024)
└── README.md


📦 Dependencies
PackageWhat It DoespandasData loading, transformation, and feature engineeringscikit-learnLabel encoding, train-test split, evaluation metricslightgbmCore gradient boosted classifiersentence-transformersSemantic item categorisation via MiniLM embeddingsnumpyNumerical operations and ranking metric computation

💰 Business Impact
The numbers are encouraging. Under conservative deployment assumptions — 10% customer acceptance rate, average add-on value of INR 100 — the system projects to generate INR 470 Crore to INR 1095 Crore in additional annual GMV at Zomato's production scale. And because the model is CPU-only, ~10MB, and serves in under 2ms, none of that requires GPU infrastructure. It's a genuinely lean system for the impact it can drive.

⚠️ Honest Limitations
We'd rather be upfront about where the model falls short than oversell it. Training covers only one month and one city, so seasonal patterns and regional cuisine diversity aren't well represented. Item-level prices aren't available as features, so the model doesn't distinguish between a INR 30 papad and a INR 300 dessert when making suggestions. And single-item orders — about 30% of all delivered orders — are currently out of scope for the recommendation logic, which is a meaningful gap we'd want to close in the next iteration.

🔭 What's Next
The most impactful near-term improvement would be training item2vec embeddings on Zomato's full order graph, replacing the label-encoded item feature with dense semantic representations that capture richer item relationships across restaurants. Beyond that: a dedicated first-add-on model for single-item carts, item-level pricing signals, and expansion to multi-city training data. Longer term, the architecture can grow into real-time streaming updates, multi-modal inputs, and a cross-vertical preference graph spanning Zomato, Blinkit, and Zomato Dining.

Developed as a Zomathon 2026 submission.