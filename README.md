# K Choice Sorter (And Approximate Sorter)

When you ask a student a question, like "Do you like this college campus photo?" The student is unable to properly preference it unless you say "...relative to this photo?". When you have many images of college campuses, this becomes a ranking problem. It's mentally straining for a student to order photos, so instead we ask a series of binary questions. Using these questions, we construct a ranking for each photo.

For an accurate ranking, a student would have to sort `n * log(n)` questions. This is generally impractical and time consuming for a student. Instead we ask a smaller number of questions and approximate the ranking of each photo. A simple way to think of this: we want to place each photo in bins such that we know the photos in each bin are next to each other in ranking. The order of the bins is known but not the order of the contents of each bin.

It is known that photos can be sorted into 2 bins in `n` time.

Each student can be asked a binary comparison OR a 4 answer question (select your favorite of these 4 photos). How can a 4 answer question be used to reduce the total number of questions asked?

# DAG Approach and Data Structure

In this approach, we construct a DAG where an edge points from any node (photo) with a lower absolute user preference to a node with an absolute higher user preference.

Each time we ask a question, we may be able to infer new questions from prior edges.

Representing this with a matrix, the algorithm is simple. Consider the following matrix **M**.

Each column represents the relationship between A and B where A is the name of the column and B is the name of the row. There are 4 different relationships. 0 = unknown, 1 = (B > A), 2 = (B = A), -1 = (B < A). Note that the diagonal is always 2, and the cells with 1 are always reflected with cells containing -1.

The matrix begins in the following form.

|         | A=1 | A=2 | A=3 | A=4 | A=5 |
| ------- | --- | --- | --- | --- | --- |
| **B=1** | 2   | 0   | 0   | 0   | 0   |
| **B=2** | 0   | 2   | 0   | 0   | 0   |
| **B=3** | 0   | 0   | 2   | 0   | 0   |
| **B=4** | 0   | 0   | 0   | 2   | 0   |
| **B=5** | 0   | 0   | 0   | 0   | 2   |

Here's a correct example matrix after the algorithm is completed.

|         | A=1 | A=2 | A=3 | A=4 | A=5 |
| ------- | --- | --- | --- | --- | --- |
| **B=1** | 2   | -1  | -1  | -1  | -1  |
| **B=2** | 1   | 2   | -1  | -1  | -1  |
| **B=3** | 1   | 1   | 2   | -1  | -1  |
| **B=4** | 1   | 1   | 1   | 2   | -1  |
| **B=5** | 1   | 1   | 1   | 1   | 2   |

You can solve with this matrix with a simple iteration.

1. M = initial matrix (shown above)
2. Find J unsolved candidates, where J is the number of photos that can be presented to the user. A simple way to do this is to find the column with the most unknowns, each unknown and the column index become candidates. Sorting the candidates by the highest number of unknowns (within a column index) yields significantly faster results. In general, the selected candidates should have the greatest expected potential for increasing the number of connections.
3. Ask question. Fill in ones where B=winner and A=$\text{loser}\_i$ where $1<i<J$, $i\neq \text{winner}$. Reflect to get -1s.
4. Iterate over each 0 in matrix at position $a,b$. If $M*{k,a}=1$ and $M*{b,k}=1$ for any $k=1...n$ then the cell $M*{a,b}=1$ else $M\*{a,b}=0$. \_This must be repeated until no operations are performed (there is
   a better solution but because n is often low, i haven't bothered to solve it)*
5. Reflect and fill in -1s.
6. 2-6 until matrix contains no unknowns (0s).

# Getting Ranking From Matrix

Convert each `0` in `M` to `0.5`, convert each `-1` to `0` and each `2` to `0`.
The sum of each row will now correspond to the best approximation of the ranking.

# Approximate Sorting

Bounding the number of iterations is a sufficient approximation. See the CollegeAI
Jupyter Notebook server for more details regarding error in relation to bounded
iterations. One can also set the stop condition to be a maximum acceptable ranking
group size, this gives a strong error guarantee.
