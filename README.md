# TakeMeter: r/soccer Discourse Classifier

3 wrong predictions for error analysis

#1 — True: hot_take, Predicted: reaction — "Thierry Henry in his post-match analysis about Ronaldo: The team needs to score, not you need to score" — This is a genuine edge case: it's a quote with emotional delivery that reads like a reaction but the content is a bold opinion.
#2 — True: news, Predicted: reaction — "German legend Franz Beckenbauer has passed away today aged 78. RIP Franz. — Fabrizio Romano" — The "RIP Franz" emotional tribute language confused the model into thinking it was a reaction rather than a news report.
#3 — True: reaction, Predicted: news — "Kansas City honors Lionel Messi after World Cup hat trick" — This reads like a factual headline but is actually capturing a celebratory community moment.
