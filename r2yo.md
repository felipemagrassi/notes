# RFC - 2023-05-28 10:22:13.578077389 -0300 -03 m=+0.117737766

tags: #rfc #pragmatic-engineer #agile #methodologies #engineering-practices

reference:

*   [Scaling Engineering Teams via Writing Things Down: RFCs](https://blog.pragmaticengineer.com/scaling-engineering-teams-via-writing-things-down-rfcs/)
*   [RFCS and Design Docs](https://blog.pragmaticengineer.com/rfcs-and-design-docs/)

***

## Problems to tackle

*   Lack of visibility on what other teams are work on
*   Tech and Architecture Debt accumulated due to different teams building things in different ways

## What is an RFC?

RFC stands for Request for Comments. It's a document that describes a problem and a proposed solution to that problem.
It's a way to communicate ideas and solutions to problems in a way that is easy to understand and easy to discuss.

## Building RFCS

*   **Plan before building something new:** whiteboarding, scrum meetings, etc.
*   **Write down the problem and the proposed solution:** once it is clear to the team, write the 'how'
*   **Have a few, selected people review the RFC and approve:** similar to a good pull request review,
    the RFC should be reviewed by a few people that are familiar with the problem and the proposed solution.
    This could be who will use the feature, people on the team, senior engineers.
*   **Once approved, send to all engineers in the company:** this is a way to share knowledge and to get feedback from other teams.
    It's also a way to avoid duplication of work.
*   **Once reviewed, implement the solution:** the RFC should be updated with the implementation details.
    This is a way to keep the RFC up to date and to have a single source of truth.

## Benefits

*   Written communication avoids meetings
*   Fewer surprises
*   Sales and Marketing can be informed about new features
*   Leveraging the knowledge of the whole company
*   Organizing thoughts and ideas
*   Specify the requirements and the implementation details

## Examples

### Uber

#### Backend

*   List of approvers
*   Abstract (what is the project about?)
*   Architecture changes
*   Service SLAs
*   Service dependencies
*   Load & performance testing
*   Multi data-center concerns
*   Security considerations
*   Testing & rollout
*   Metrics & monitoring
*   Customer support considerations

#### Mobile/Web

*   List of approvers
*   Abstract (what is the project about?)
*   UI & UX
*   Architecture changes
*   Network interactions detailed
*   Library dependencies
*   Security concerns
*   Testing & rollout
*   Analytics
*   Customer support considerations
*   Accessibility

### Design Docs

### RDA
