# lifeforecast

LifeForecast MVP (Version 1)

-- Enable UUID extension

create extension if not exists "uuid-ossp";



-- Profiles table

create table if not exists public.profiles (

&nbsp; id uuid primary key references auth.users(id) on delete cascade,

&nbsp; full\_name text,

&nbsp; timezone text default 'America/New\_York',

&nbsp; created\_at timestamptz default now(),

&nbsp; updated\_at timestamptz default now()

);



-- Identity Canvas (1 per user)

create table if not exists public.identity\_canvas (

&nbsp; id uuid primary key default uuid\_generate\_v4(),

&nbsp; user\_id uuid not null references auth.users(id) on delete cascade,

&nbsp; mission text,

&nbsp; vision text,

&nbsp; values text,

&nbsp; customer text,

&nbsp; offer text,

&nbsp; differentiator text,

&nbsp; revenue\_streams jsonb default '\[]'::jsonb,

&nbsp; channels jsonb default '\[]'::jsonb,

&nbsp; key\_activities jsonb default '\[]'::jsonb,

&nbsp; created\_at timestamptz default now(),

&nbsp; updated\_at timestamptz default now(),

&nbsp; unique(user\_id)

);



-- SWOT (1 per user)

create table if not exists public.swot (

&nbsp; id uuid primary key default uuid\_generate\_v4(),

&nbsp; user\_id uuid not null references auth.users(id) on delete cascade,

&nbsp; strengths jsonb default '\[]'::jsonb,

&nbsp; weaknesses jsonb default '\[]'::jsonb,

&nbsp; opportunities jsonb default '\[]'::jsonb,

&nbsp; threats jsonb default '\[]'::jsonb,

&nbsp; top\_focus jsonb default '\[]'::jsonb,

&nbsp; created\_at timestamptz default now(),

&nbsp; updated\_at timestamptz default now(),

&nbsp; unique(user\_id)

);



-- Goals (many per user)

create table if not exists public.goals (

&nbsp; id uuid primary key default uuid\_generate\_v4(),

&nbsp; user\_id uuid not null references auth.users(id) on delete cascade,

&nbsp; title text not null,

&nbsp; description text,

&nbsp; specific text,

&nbsp; measurable text,

&nbsp; achievable text,

&nbsp; relevant text,

&nbsp; time\_bound text,

&nbsp; start\_date date,

&nbsp; due\_date date,

&nbsp; status text default 'planned' check (status in ('planned','active','done')),

&nbsp; priority int default 2 check (priority between 1 and 3),

&nbsp; progress int default 0 check (progress between 0 and 100),

&nbsp; created\_at timestamptz default now(),

&nbsp; updated\_at timestamptz default now()

);



-- Milestones (many per goal)

create table if not exists public.milestones (

&nbsp; id uuid primary key default uuid\_generate\_v4(),

&nbsp; user\_id uuid not null references auth.users(id) on delete cascade,

&nbsp; goal\_id uuid not null references public.goals(id) on delete cascade,

&nbsp; title text not null,

&nbsp; due\_date date,

&nbsp; status text default 'open' check (status in ('open','done')),

&nbsp; created\_at timestamptz default now(),

&nbsp; updated\_at timestamptz default now()

);



-- Weekly reviews (optional MVP, can build later)

create table if not exists public.weekly\_reviews (

&nbsp; id uuid primary key default uuid\_generate\_v4(),

&nbsp; user\_id uuid not null references auth.users(id) on delete cascade,

&nbsp; week\_start\_date date not null,

&nbsp; wins text,

&nbsp; lessons text,

&nbsp; next\_week\_focus text,

&nbsp; created\_at timestamptz default now(),

&nbsp; updated\_at timestamptz default now(),

&nbsp; unique(user\_id, week\_start\_date)

);



-- RLS ON

alter table public.profiles enable row level security;

alter table public.identity\_canvas enable row level security;

alter table public.swot enable row level security;

alter table public.goals enable row level security;

alter table public.milestones enable row level security;

alter table public.weekly\_reviews enable row level security;



-- Policies: user can only access their own rows

create policy "profiles\_select\_own" on public.profiles

for select using (auth.uid() = id);



create policy "profiles\_update\_own" on public.profiles

for update using (auth.uid() = id);



create policy "profiles\_insert\_own" on public.profiles

for insert with check (auth.uid() = id);



create policy "identity\_select\_own" on public.identity\_canvas

for select using (auth.uid() = user\_id);



create policy "identity\_upsert\_own" on public.identity\_canvas

for all using (auth.uid() = user\_id) with check (auth.uid() = user\_id);



create policy "swot\_select\_own" on public.swot

for select using (auth.uid() = user\_id);



create policy "swot\_upsert\_own" on public.swot

for all using (auth.uid() = user\_id) with check (auth.uid() = user\_id);



create policy "goals\_select\_own" on public.goals

for select using (auth.uid() = user\_id);



create policy "goals\_crud\_own" on public.goals

for all using (auth.uid() = user\_id) with check (auth.uid() = user\_id);



create policy "milestones\_select\_own" on public.milestones

for select using (auth.uid() = user\_id);



create policy "milestones\_crud\_own" on public.milestones

for all using (auth.uid() = user\_id) with check (auth.uid() = user\_id);



create policy "weekly\_reviews\_select\_own" on public.weekly\_reviews

for select using (auth.uid() = user\_id);



create policy "weekly\_reviews\_crud\_own" on public.weekly\_reviews

for all using (auth.uid() = user\_id) with check (auth.uid() = user\_id);

