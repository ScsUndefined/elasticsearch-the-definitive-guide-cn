# Aggregations

Until this point, this book has been dedicated to search. With search, we have a query and we want to find a subset of documents that match the query. We are looking for the proverbial needle(s) in the haystack.

With aggregations, we zoom out to get an overview of our data. Instead of looking for individual documents, we want to analyze and summarize our complete set of data:

How many needles are in the haystack?
What is the average length of the needles?
What is the median length of the needles, broken down by manufacturer?
How many needles were added to the haystack each month?
Aggregations can answer more subtle questions too:

What are your most popular needle manufacturers?
Are there any unusual or anomalous clumps of needles?
Aggregations allow us to ask sophisticated questions of our data. And yet, while the functionality is completely different from search, it leverages the same data-structures. This means aggregations execute quickly and are near real-time, just like search.

This is extremely powerful for reporting and dashboards. Instead of performing rollups of your data (that crusty Hadoop job that takes a week to run), you can visualize your data in real time, allowing you to respond immediately. Your report changes as your data changes, rather than being pre-calculated, out of date and irrelevant.

Finally, aggregations operate alongside search requests. This means you can both search/filter documents and perform analytics at the same time, on the same data, in a single request. And because aggregations are calculated in the context of a user’s search, you’re not just displaying a count of four-star hotels—you’re displaying a count of four-star hotels that match their search criteria.

Aggregations are so powerful that many companies have built large Elasticsearch clusters solely for analytics.