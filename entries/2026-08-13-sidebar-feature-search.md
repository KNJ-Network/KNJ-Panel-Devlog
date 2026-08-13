# Phase 74 - A Search Box That Lives Where You're Already Looking

A small follow-up to the dashboard work: a feature search, added to both the account and
controller sidebars.

## Why the sidebar, not the top of the page

Many hosting panels put search in a banner across the top of the content area. That splits your
attention: you look at the sidebar to navigate, then jump your eyes elsewhere to search. Since the
point is to find things in this list faster, the box sits directly above the list it searches, at
the very top of the sidebar.

## How it works

Type, and the navigation filters in place: matching items stay, non-matching items and empty groups
disappear as you type, with no page reload. Press `/` from anywhere to jump into the box, Enter to
go to the first match, and Escape to clear. It's a small self-contained script with no framework
dependency, and it works on both sidebars because both render from the same navigation data.
