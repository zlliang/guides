# English Writing

A comprehensive guide to writing clearly, concisely, and idiomatically in English. This guide covers technical essays, personal reflections, and blog posts, drawing inspiration from influential writers like Paul Graham, Bob Nystrom, Simon Willison, Mitchell Hashimoto, and Alex Kladov.

## Quick Reference

### Core Principles

- **Clarity First**: Say what you mean. Avoid ambiguity.
- **Cut Ruthlessly**: Every word must earn its place.
- **Show, Don't Tell**: Use examples, not abstractions.
- **Write for Humans**: Conversational beats formal.
- **Revise, Revise, Revise**: Writing is rewriting.

### Common Fixes

```
❌ utilize → ✓ use
❌ in order to → ✓ to
❌ it is important to note that → ✓ note that / delete
❌ at this point in time → ✓ now
❌ due to the fact that → ✓ because
```

### Structure Template

```
1. Hook (grab attention)
2. Context (set the scene)
3. Main point (state your thesis)
4. Evidence/Examples (support your point)
5. Counterarguments (acknowledge alternatives)
6. Conclusion (tie it together)
```

## Introduction

### What This Guide Covers

This guide teaches you to write:

1. **Technical Essays**: Explaining complex ideas clearly (programming, systems, tools)
2. **Personal Reflections**: Sharing experiences and lessons learned
3. **Blog Posts**: Engaging readers with ideas, stories, and insights
4. **Professional Communication**: Emails, documentation, proposals

### What Makes Good Writing?

Good writing is:
- **Clear**: Readers understand on first reading
- **Concise**: No wasted words
- **Correct**: Grammar and facts are accurate
- **Engaging**: Readers want to keep reading
- **Authentic**: Sounds like a real person wrote it

### Who This Guide Is For

- **Developers** writing technical blogs
- **Researchers** sharing findings
- **Entrepreneurs** communicating vision
- **Anyone** who wants to write better

## Fundamentals

### Clarity

Clarity means your reader understands exactly what you mean.

**Write Simple Sentences**:
```
❌ The implementation of the algorithm, which was designed to optimize 
   performance, resulted in significant improvements.
   
✓ We implemented an algorithm that made the system 3x faster.
```

**Use Concrete Words**:
```
❌ We need to enhance our velocity.
✓ We need to ship faster.
```

**Define Technical Terms**:
```
❌ Use memoization to avoid recomputing.

✓ Use memoization—caching results of function calls—to avoid recomputing.
```

**One Idea Per Sentence**:
```
❌ The function validates input, which should be a string, and returns 
   true if valid, otherwise false, and logs errors.

✓ The function validates input strings. It returns true if valid, 
   false otherwise. It also logs any errors.
```

### Conciseness

Every word must serve a purpose.

**Cut Filler Words**:
```
❌ It should be noted that...
❌ It is important to mention that...
❌ Basically...
❌ Actually...

✓ [Delete these phrases]
```

**Prefer Active Voice**:
```
❌ The bug was fixed by the team.
✓ The team fixed the bug.

❌ The decision was made to refactor.
✓ We decided to refactor.
```

**Eliminate Redundancy**:
```
❌ completely finished → finished
❌ final result → result
❌ past history → history
❌ advance planning → planning
❌ free gift → gift
```

**Shorten Phrases**:
```
❌ at the present time → now
❌ in the event that → if
❌ for the purpose of → to
❌ with regard to → about
❌ a number of → several / many
```

### Correctness

**Grammar Basics**:

*Subject-Verb Agreement*:
```
✓ The system works well.
❌ The system work well.

✓ The developers are ready.
❌ The developers is ready.
```

*Pronoun Clarity*:
```
❌ When you update the cache, it might fail.
   (What might fail? The cache or the update?)

✓ When you update the cache, the update might fail.
```

*Parallel Structure*:
```
❌ I like reading, to write, and code.
✓ I like reading, writing, and coding.
✓ I like to read, write, and code.
```

**Common Mistakes**:

*Its vs It's*:
```
✓ The system has its own cache. (possessive)
✓ It's a simple fix. (it is)
```

*Their / They're / There*:
```
✓ Their code is clean. (possessive)
✓ They're shipping today. (they are)
✓ Put it there. (location)
```

*Affect vs Effect*:
```
✓ The bug affects performance. (verb)
✓ The effect is noticeable. (noun)
```

## Style and Voice

### Conversational Writing

Write like you talk (but edited).

**Use Contractions**:
```
Formal: I do not think it is a good idea.
Better: I don't think it's a good idea.
```

**Address the Reader**:
```
Distant: One might consider using a hash table.
Better: You might consider using a hash table.
Best: Consider using a hash table.
```

**Ask Questions**:
```
Why does this matter? Because performance is critical.
How do we solve this? Let's look at three approaches.
```

### Finding Your Voice

Your voice is what makes your writing unique.

**Be Yourself**:
- Don't try to sound smarter than you are
- Use words you'd actually say
- Share your real opinions
- Admit what you don't know

**Show Personality**:
```
Bland: The approach has some advantages.
Better: This approach is clever, if a bit hacky.

Bland: The code needs improvement.
Better: This code makes me sad.
```

**Use Humor (When Appropriate)**:
```
"I spent three hours debugging this. The bug was a missing comma. 
I'm now a professional comma hunter."
```

### Tone

Match your tone to your audience and purpose.

**Technical Blog** (Paul Graham style):
- Conversational but thoughtful
- Challenge conventional wisdom
- Use analogies from everyday life

**Tutorial** (Bob Nystrom style):
- Encouraging and supportive
- Step-by-step with examples
- Anticipate confusion

**Personal Reflection** (Mitchell Hashimoto style):
- Honest and vulnerable
- Share both success and failure
- Connect personal to universal

## Technical Writing

### Explaining Complex Ideas

**Start Simple**:
```
Don't: "Implement a self-balancing binary search tree."

Do: "Let's build a phonebook. Each person has a name and number. 
     We want to find people quickly..."
```

**Use Analogies**:
```
"A hash table is like a filing cabinet. Each drawer (bucket) has a 
label (hash), and you can go straight to the right drawer instead 
of searching every drawer."
```

**Build Up Gradually** (Bob Nystrom's approach):
1. Show the simple case
2. Explain why it fails
3. Add complexity incrementally
4. Show code at each step

**Example: Explaining Recursion**:
```
Bad: "A function that calls itself with modified parameters."

Good:
"Imagine you're in a maze. At each intersection, you have choices.
 
 You could try to remember every turn. Or you could:
 1. Try the left path
 2. If you hit a dead end, come back and try right
 3. Repeat at each intersection
 
 This is recursion: solving a problem by solving smaller versions 
 of the same problem."
```

### Code Examples

**Keep Them Simple**:
```python
# ❌ Too much at once
class ComplexCacheSystem:
    def __init__(self, size, eviction_policy, serializer):
        self.cache = LRUCache(size)
        self.policy = eviction_policy
        ...

# ✓ Start minimal
cache = {}
cache['key'] = 'value'
result = cache.get('key')
```

**Show Before and After**:
```python
# Before: Slow
for item in items:
    if is_valid(item):
        process(item)

# After: Fast (with explanation why)
valid_items = [item for item in items if is_valid(item)]
for item in valid_items:
    process(item)
```

**Explain Non-Obvious Parts**:
```python
# Check if power of 2 using bit trick
# (powers of 2 have exactly one bit set)
if n & (n - 1) == 0:
    return True
```

### Structure for Technical Posts

**1. Hook** (Simon Willison style):
```
"I just spent a week debugging a performance issue. 
 The fix was a one-line change. Here's what I learned."
```

**2. Context**:
- What problem were you solving?
- Why does it matter?
- What had you tried?

**3. Solution**:
- Walk through your approach
- Show code/examples
- Explain trade-offs

**4. Lessons**:
- What would you do differently?
- When does this apply?
- What's still unsolved?

## Personal Writing

### Essays and Reflections

**Start with a Specific Moment**:
```
Don't: "Building a startup is hard."

Do: "At 2 AM, staring at our rapidly depleting bank account, 
     I realized we had three weeks of runway left."
```

**Be Honest**:
```
Don't: "We successfully pivoted our strategy."

Do: "Our product failed. Nobody wanted it. We were wrong about 
     everything except one tiny feature users loved."
```

**Connect Personal to Universal**:
```
"My experience isn't unique. Every founder I know has faced this 
exact moment: when reality doesn't match the pitch deck."
```

### Storytelling

**Show, Don't Tell**:
```
Tell: "I was nervous about the presentation."

Show: "My hands shook as I opened the laptop. I'd rehearsed this 
       a hundred times, but my mind went blank as fifty faces 
       turned toward me."
```

**Use Dialogue**:
```
"You're overthinking this," my cofounder said.
"Maybe. But what if—"
"No. Ship it."
```

**Create Tension**:
```
Good stories have:
- A problem (conflict)
- Rising stakes (complications)
- A turning point (decision/discovery)
- Resolution (outcome and reflection)
```

### Finding Ideas

**Write What You're Learning**:
- Document your struggles
- Explain to your past self
- Share your breakthroughs

**Write What You Disagree With**:
- Challenge conventional wisdom
- Explain why everyone is wrong
- Propose alternatives

**Write What You Wish Existed**:
- The tutorial you needed
- The perspective missing from discourse
- The connection nobody made

## Studying Great Writers

### Paul Graham

**Characteristics**:
- Conversational yet intellectual
- Challenges assumptions
- Uses simple words for complex ideas
- Short paragraphs, one idea each

**Technique: Contrarian Thinking**:
```
"Most people think X. They're wrong because Y. 
 Here's what actually matters: Z."
```

**Example Structure**:
1. Provocative claim
2. Counterintuitive insight
3. Historical/practical evidence
4. Broader implications

**Learn From**:
- [How to Do Great Work](http://www.paulgraham.com/greatwork.html)
- [Write Like You Talk](http://www.paulgraham.com/talk.html)

### Bob Nystrom

**Characteristics**:
- Incredibly clear explanations
- Builds complexity gradually
- Friendly, encouraging tone
- Excellent use of diagrams

**Technique: Layered Teaching**:
```
1. Show the simple version
2. Point out where it breaks
3. Add one fix
4. Repeat until complete
```

**Example Approach** (from Crafting Interpreters):
- "Here's a calculator"
- "Now let's add variables"
- "Now let's add functions"
- Each step builds on the last

**Learn From**:
- [What color is your function?](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)
- [Crafting "Crafting Interpreters"](https://journal.stuffwithstuff.com/2020/04/05/crafting-crafting-interpreters/)
- [Stupid Dog](https://journal.stuffwithstuff.com/2022/02/13/stupid-dog/)
- [Crafting Interpreters](https://craftinginterpreters.com/)

### Simon Willison

**Characteristics**:
- Practical and timely
- Heavy use of examples
- Links to sources
- "Here's what I built" approach

**Technique: Show Your Work**:
```
1. Here's what I needed to do
2. Here's what I tried
3. Here's what I learned
4. Here's the code/tool
```

**Example Pattern**:
- Identify a problem
- Build a solution (often a small tool)
- Write about the process
- Share code and demos

**Learn From**:
- Blog posts about AI, tools, datasette
- TIL (Today I Learned) posts
- Open source documentation

### Alex Kladov (matklad)

**Characteristics**:
- Deep technical insights
- Strong opinions, clearly stated
- Focuses on fundamentals
- Excellent at explaining "why"

**Technique: Principled Arguments**:
```
1. State the principle
2. Show concrete examples
3. Explain trade-offs
4. Recommend approach
```

**Example Topics**:
- Why certain patterns matter
- How systems should be designed
- What makes code maintainable

**Learn From**:
- Blog posts on Rust, IDEs, architecture
- Focus on "boring solutions"
- Emphasis on simplicity

### Mitchell Hashimoto

**Characteristics**:
- Vulnerable and honest
- Balances technical and personal
- Long-form deep dives
- Reflective about journey

**Technique: Integrated Learning**:
```
1. Share the challenge (personal or technical)
2. Document the journey
3. Extract lessons
4. Connect to broader themes
```

**Example Approach**:
- Building something hard
- Facing setbacks
- Learning and adapting
- Sharing both code and emotions

**Learn From**:
- Posts about building HashiCorp
- Personal growth reflections
- Technical deep dives

## Practical Techniques

### Opening Hooks

**Start with Action**:
```
"I deleted the production database. By accident."
```

**Start with a Question**:
```
"Why do we keep building the same thing over and over?"
```

**Start with Surprising Fact**:
```
"90% of performance bugs come from one type of mistake."
```

**Start with Conflict**:
```
"Everyone says to use microservices. Everyone is wrong."
```

### Transitions

**Between Paragraphs**:
```
But here's the catch...
This leads to a problem...
Let's see how this works...
Here's what I mean...
```

**Between Sections**:
```
So far we've covered X. Now let's look at Y.
That's the theory. Here's the practice.
That solves A, but what about B?
```

### Conclusions

**Summarize Key Points**:
```
To recap:
- Point 1
- Point 2  
- Point 3
```

**Call to Action**:
```
Try this yourself. Build something. Break something. Learn.
```

**Open a Door**:
```
This solves X, but leaves Y open. That's what we'll explore next.
```

**Personal Reflection**:
```
Writing this taught me that...
I'm still learning, but here's what I know now...
```

## Common Mistakes

### Overcomplicating

**Don't Use Big Words to Sound Smart**:
```
❌ Utilize → ✓ Use
❌ Commence → ✓ Start
❌ Terminate → ✓ End
❌ Facilitate → ✓ Help
```

**Don't Write Long Sentences**:
```
❌ The system, which was designed to handle large-scale data 
   processing tasks, utilizes a distributed architecture that 
   allows it to scale horizontally across multiple nodes.

✓ The system processes large amounts of data. It uses a 
   distributed architecture. This lets it scale across nodes.
```

### Being Vague

**Provide Specifics**:
```
❌ The performance improved significantly.
✓ Response time dropped from 2 seconds to 200ms.

❌ It's much faster now.
✓ It's 10x faster.
```

### Over-Explaining

**Trust Your Reader**:
```
❌ First, we need to import the library, which is a collection 
   of pre-written code that provides functionality...

✓ First, import the library:
   import requests
```

### Hiding Behind Passive Voice

**Take Ownership**:
```
❌ Mistakes were made.
✓ We made mistakes.

❌ The decision was made to proceed.
✓ We decided to proceed.
```

## Writing Process

### Drafting

**First Draft Rules**:
1. Don't edit while writing
2. Write badly if needed
3. Skip parts you're stuck on
4. Just get words down

**Free Writing**:
```
Set a timer for 20 minutes.
Write without stopping.
Don't fix typos.
Don't delete anything.
```

### Revising

**Multiple Passes**:

**Pass 1: Structure**
- Does the flow make sense?
- Are sections in the right order?
- Is anything missing?

**Pass 2: Clarity**
- Can I simplify sentences?
- Are examples clear?
- Will readers understand?

**Pass 3: Conciseness**
- Can I cut this word/sentence/paragraph?
- Is every word necessary?
- Am I repeating myself?

**Pass 4: Polish**
- Fix grammar
- Check facts
- Verify code examples

### Getting Feedback

**Questions to Ask Reviewers**:
- What confused you?
- What could I cut?
- What do you want more of?
- What's the main point?

**Act on Feedback**:
```
If one person is confused, rewrite.
If multiple people miss the point, restructure.
If everyone says it's too long, cut 30%.
```

## Building Habits

### Write Regularly

**Daily Practice**:
- Morning pages (3 pages, stream of consciousness)
- Write for 30 minutes daily
- Publish weekly (blog, newsletter)

**Small Wins**:
- TIL (Today I Learned) posts
- Tweet threads
- Comments explaining code

### Read Actively

**Analyze Good Writing**:
1. How did they hook you?
2. How did they structure it?
3. What made it clear?
4. What could be better?

**Copy to Learn**:
- Retype paragraphs you admire
- Notice sentence structure
- Feel the rhythm

### Experiment

**Try Different Formats**:
- How-to guides
- Personal essays
- Technical deep dives
- Quick tips
- Story-driven posts

**Try Different Styles**:
- Formal vs casual
- Long vs short
- Technical vs accessible
- Serious vs humorous

## Exercises

### Clarity Exercise

Take this paragraph and make it clearer:
```
The implementation of the new feature, which was requested by 
several users, has been completed and deployed to production, 
resulting in improved user experience metrics.
```

**Sample Revision**:
```
We shipped the new feature users requested. User satisfaction 
improved by 20%.
```

### Conciseness Exercise

Cut this 100-word paragraph to 50 words:
```
In today's fast-paced technological landscape, it is increasingly 
important to note that software development practices are constantly 
evolving. It should be mentioned that developers need to stay up to 
date with the latest trends and best practices. At this point in 
time, we can observe that continuous learning is essential for 
success in the field.
```

**Sample Revision (43 words)**:
```
Software development evolves constantly. Developers must stay 
current with trends and best practices. Continuous learning 
is essential for success.
```

### Voice Exercise

Rewrite this in three different tones:
```
Original: "Error handling is important for robust applications."

Casual: "If your app crashes every time something goes wrong, 
         users will hate it."

Academic: "Robust error handling is essential for production-grade 
           software systems."

Storytelling: "I learned about error handling the hard way: at 3 AM, 
               when the CEO called about the site being down."
```

## Resources

### Books

**On Writing Well** by William Zinsser
- Classic guide to non-fiction writing
- Emphasizes clarity and simplicity
- Timeless principles

**The Elements of Style** by Strunk and White
- Concise rules of usage
- Grammar fundamentals
- Required reading

**On Writing** by Stephen King
- Writing craft and process
- Honest about the work
- Inspiring and practical

**Bird by Bird** by Anne Lamott
- Writing as a practice
- Overcoming perfectionism
- "Shitty first drafts"

**Several Short Sentences About Writing** by Verlyn Klinkenborg
- Unique approach to sentences
- Breaking bad habits
- Fresh perspective

### Online Resources

**Essays on Writing**:
- [Write Like You Talk](http://www.paulgraham.com/talk.html) - Paul Graham
- [Writing, Briefly](http://www.paulgraham.com/writing44.html) - Paul Graham  
- [How to Write Technical Posts](https://reasonablypolymorphic.com/blog/writing-technical-posts/) - Sandy Maguire

**Style Guides**:
- [Google Developer Documentation Style Guide](https://developers.google.com/style)
- [Microsoft Writing Style Guide](https://learn.microsoft.com/en-us/style-guide/welcome/)
- [Hemingway Editor](http://www.hemingwayapp.com/) - Readability tool

**Communities**:
- Write.as - Minimalist blogging
- Dev.to - Developer community
- Hacker News - Tech discourse
- IndieHackers - Founder stories

### Tools

**Writing**:
- [Grammarly](https://www.grammarly.com/) - Grammar and clarity
- [Hemingway App](http://www.hemingwayapp.com/) - Readability
- [LanguageTool](https://languagetool.org/) - Grammar checker

**Publishing**:
- [Ghost](https://ghost.org/) - Blogging platform
- [Substack](https://substack.com/) - Newsletter platform
- [Medium](https://medium.com/) - Publication platform

**Reading**:
- RSS readers for following writers
- Pocket for saving articles
- Readwise for highlights

## Final Thoughts

### Writing is a Craft

You get better by doing:
- Write regularly
- Read actively
- Revise ruthlessly
- Publish bravely

### Your Voice Matters

Don't try to sound like someone else:
- Write what you know
- Write what you care about
- Write what only you can write

### Start Simple

You don't need:
- Perfect grammar
- Big words
- Complete understanding

You do need:
- Something to say
- Willingness to revise
- Courage to share

### Keep Going

> "The scariest moment is always just before you start." 
> — Stephen King

Write that first draft.
Publish that post.
Start that blog.

Your writing will improve.
Your voice will emerge.
Your ideas will matter.

Now go write.
