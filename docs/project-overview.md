# SAME DAY. — Project Overview

## What it is

A single-page birthday calendar. People can find a month and day, see entries for that date, and add their own name and birthday.

## How it works today

- The page is one HTML file containing the layout, styling, and JavaScript.
- JavaScript builds the calendar, searches dates, filters by birth year, and calculates summary stats.
- It creates 2,400 sample people in the browser; these are demonstration data, not real users.
- New entries and removals are saved in that browser's `localStorage`. Other people and devices cannot see them.
- Form checks, duplicate checks, and the submission limit run in the browser only.

## What that means

The page is currently a local demo, not a shared live service. Clearing browser data can remove saved entries. The page says submitted names and dates are public, so a live version should explain that clearly.
