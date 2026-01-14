<overview>
Contextual questions for intake, organized by purpose. Use questions to clarify requirements.
</overview>

<universal_questions>

<topic_identifier>
When topic not obvious from description:
"What topic/feature is this for? (used for file naming)"
</topic_identifier>

<chain_reference>
When existing research/plan files found:
"Should this prompt reference any existing research or plans? (Select multiple if needed)"
- "{file1}" - Found in .prompts/{folder1}/
- "{file2}" - Found in .prompts/{folder2}/
- "None" - Start fresh without referencing existing files
</chain_reference>

</universal_questions>

<do_questions>

<artifact_type>
When unclear what's being created:
"What are you creating?"
- "Code/feature" - Software implementation
- "Document/content" - Written material, documentation
- "Design/spec" - Architecture, wireframes, specifications
- "Configuration" - Config files, infrastructure setup
</artifact_type>

<scope_completeness>
When level of polish unclear:
"What level of completeness?"
- "Production-ready" - Ship to users, needs polish and tests
- "Working prototype" - Functional but rough edges acceptable
- "Proof of concept" - Minimal viable demonstration
</scope_completeness>

<approach_patterns>
When implementation approach unclear:
"Any specific patterns or constraints?"
- "Follow existing patterns" - Match current codebase style
- "Best practices" - Modern, recommended approaches
- "Specific requirement" - I have a constraint to specify
</approach_patterns>

<testing_requirements>
When verification needs unclear:
"What testing is needed?"
- "Full test coverage" - Unit, integration, e2e tests
- "Core functionality" - Key paths tested
- "Manual verification" - No automated tests required
</testing_requirements>

<integration_points>
For features that connect to existing code:
"How does this integrate with existing code?"
- "New module" - Standalone, minimal integration
- "Extends existing" - Adds to current implementation
- "Replaces existing" - Replaces current implementation
</integration_points>

</do_questions>

<plan_questions>

<plan_purpose>
What the plan leads to:
"What is this plan leading to?"
- "Implementation" - Break down how to build something
- "Decision" - Weigh options, choose an approach
- "Process" - Define workflow or methodology
</plan_purpose>

<plan_format>
How to structure the output:
"What format works best?"
- "Phased roadmap" - Sequential stages with milestones
- "Checklist/tasks" - Actionable items to complete
- "Decision framework" - Criteria, trade-offs, recommendation
</plan_format>

<constraints>
What limits the plan:
"What constraints should the plan consider?"
- "Technical" - Stack limitations, dependencies, compatibility
- "Resources" - Team capacity, expertise available
- "Requirements" - Must-haves, compliance, standards
</constraints>

<granularity>
Level of detail needed:
"How detailed should the plan be?"
- "High-level phases" - Major milestones, flexible execution
- "Detailed tasks" - Specific actionable items
- "Prompt-ready" - Each phase is one prompt to execute
</granularity>

<dependencies>
What exists vs what needs creation:
"What already exists?"
- "Greenfield" - Starting from scratch
- "Existing codebase" - Building on current code
- "Research complete" - Findings ready to plan from
</dependencies>

</plan_questions>

<research_questions>

<research_depth>
How comprehensive:
"How deep should the research go?"
- "Overview" - High-level understanding, key concepts
- "Comprehensive" - Detailed exploration, multiple perspectives
- "Exhaustive" - Everything available, edge cases included
</research_depth>

<source_priorities>
Where to look:
"What sources should be prioritized?"
- "Official docs" - Primary sources, authoritative references
- "Community" - Blog posts, tutorials, real-world examples
- "Current/latest" - 2024-2025 sources, cutting edge
</source_priorities>

<output_format>
How to present findings:
"How should findings be structured?"
- "Summary with key points" - Concise, actionable takeaways
- "Detailed analysis" - In-depth with examples and comparisons
- "Reference document" - Organized for future lookup
</output_format>

<research_focus>
When topic is broad:
"What aspect is most important?"
- "How it works" - Concepts, architecture, internals
- "How to use it" - Patterns, examples, best practices
- "Trade-offs" - Pros/cons, alternatives, comparisons
</research_focus>

<evaluation_criteria>
For comparison research:
"What criteria matter most for evaluation?"
- "Performance" - Speed, scalability, efficiency
- "Developer experience" - Ease of use, documentation, community
- "Security" - Vulnerabilities, compliance, best practices
- "Cost" - Pricing, resource usage, maintenance
</evaluation_criteria>

</research_questions>

<refine_questions>

<target_selection>
When multiple outputs exist:
"Which output should be refined?"
- "{file1}" - In .prompts/{folder1}/
- "{file2}" - In .prompts/{folder2}/
</target_selection>

<feedback_type>
What kind of improvement:
"What needs improvement?"
- "Deepen analysis" - Add more detail, examples, or rigor
- "Expand scope" - Cover additional areas or topics
- "Correct errors" - Fix factual mistakes or outdated info
- "Restructure" - Reorganize for clarity or usability
</feedback_type>

<specific_feedback>
After type selected, gather details:
"What specifically should be improved?"
</specific_feedback>

<preservation>
What to keep:
"What's working well that should be kept?"
- "Structure" - Keep the overall organization
- "Recommendations" - Keep the conclusions
- "Code examples" - Keep the implementation patterns
- "Everything except feedback areas" - Only change what's specified
</preservation>

</refine_questions>

<question_rules>
- Only ask about genuine gaps - don't ask what's already stated
- 2-4 questions max per round - avoid overwhelming
- Each option needs description - explain implications
- Prefer options over free-text - when choices are knowable
- User can always select "Other" - for custom input
- Route by purpose - use purpose-specific questions after primary gate
</question_rules>