
A) Minimum features (≥8) — implement these

User registration / login / logout (Auth)

User profile CRUD (edit profile, avatar)

Event CRUD (create/edit/delete events — admin)

Event listing / search / filter / pagination (Read)

Ticket booking / reservation CRUD (create, view, cancel)

Event categories CRUD (foreign-key relation)

Comments / reviews on events (create/read/delete)

Admin dashboard (view stats: events, bookings, users)

Notifications (email or in-app) — optional extra

File upload for event images — optional extra

(You only need 8 + auth; this gives 10 so safe.)

B) Database design (conceptual + physical summary)

Entities (main):

users (id, name, email, password, role, created_at, updated_at)

profiles (id, user_id FK -> users.id, phone, address, avatar_url)

categories (id, name, slug)

events (id, category_id FK -> categories.id, title, description, location, start_time, end_time, capacity, price, image_url, created_by -> users.id)

tickets (id, user_id FK -> users.id, event_id FK -> events.id, status, purchased_at, qty)

comments (id, user_id FK -> users.id, event_id FK -> events.id, body, rating, created_at)

logs/notifications (optional)

Foreign keys examples:

events.category_id → categories.id

tickets.event_id → events.id

tickets.user_id → users.id

profiles.user_id → users.id

Physical: add indexes on foreign keys and on events(start_time), events(title) for search.

C) Laravel implementation plan (step-by-step commands & snippets)

Assume Laravel 10+.

New project

    composer create-project laravel/laravel eventapp
    cd eventapp
    php artisan serve


Auth scaffolding (use Breeze/Jetstream or make-simple)

    composer require laravel/breeze --dev
    php artisan breeze:install
    npm install && npm run dev
    php artisan migrate


(If you don’t use Breeze, implement simple auth with Laravel built-in auth routes/controllers.)

Create models + migrations + controllers + resource routes

    php artisan make:model Category -mcr   
    php artisan make:model Event -mcr
    php artisan make:model Ticket -mcr
    php artisan make:model Profile -m -f    
    php artisan make:model Comment -mcr


Example migration (events)

    Schema::create('events', function (Blueprint $table) {
        $table->id();
        $table->foreignId('category_id')->constrained()->onDelete('cascade');
        $table->foreignId('created_by')->constrained('users')->onDelete('cascade');
        $table->string('title');
        $table->text('description')->nullable();
        $table->string('location')->nullable();
        $table->dateTime('start_time');
        $table->dateTime('end_time')->nullable();
        $table->integer('capacity')->default(0);
        $table->decimal('price', 10, 2)->default(0);
        $table->string('image_url')->nullable();
        $table->timestamps();
    });          

Example relations (models)

    class Event extends Model {
        protected $fillable = ['category_id','created_by','title','description','location','start_time','end_time','capacity','price','image_url'];
        public function category() { return $this->belongsTo(Category::class); }
        public function tickets() { return $this->hasMany(Ticket::class); }
        public function creator() { return $this->belongsTo(User::class,'created_by'); }
        public function comments() { return $this->hasMany(Comment::class); }
    }


Routes (web.php)

    Route::get('/', [EventController::class, 'index'])->name('home');
    
    Route::middleware('auth')->group(function(){
        Route::resource('events','EventController'); // restrict create/edit to admin in Controller or middleware
        Route::resource('tickets','TicketController')->only(['index','store','destroy','show']);
        Route::post('events/{event}/comments',[CommentController::class,'store'])->name('events.comments.store');
        Route::get('dashboard',[AdminController::class,'index'])->middleware('can:isAdmin')->name('admin.dashboard');
    });


Controller skeleton (EventController)

1) index() list with search & pagination.

2) create() shows form.
   
3) store(Request $r) validate, store.

4) show($id) show event + comments + ticket button.
   
6) edit(), update(), destroy() for admin.

Validation example

    $request->validate([
     'title'=>'required|string|max:255',
     'start_time'=>'required|date',
     'end_time'=>'nullable|date|after_or_equal:start_time',
     'capacity'=>'required|integer|min:0',
    ]);


Ticket booking (basic)

On purchase: check tickets count vs events.capacity

Create ticket record, decrement available seats (or compute availability from count)

File uploads
Use Storage::putFile('public/events',$request->file('image')) and save URL.

Seeders & factories
Create seeders for categories, sample events, test users.

Unit/Feature tests
Write feature tests for auth, event creation, booking.

D) What to include in the report (structure + content)

Make a PDF (Report.PDF) containing:

Title page (project title, members, IDs)

Abstract / Objective (short)

Requirements mapping (list course requirements and how you satisfy them)

Database design

ER diagram (conceptual)

Physical schema (table list + sample SQL / migration snippets)

Implemented features

List each feature (the 8+), short explanation (~2–4 lines), mention routes/controllers used.

Screenshots & explanation

For each feature: 1 screenshot + caption + short notes (what it does, inputs, outputs).

Work division

Clear percentage and bullet list of each person’s contributions.

Deployment

Link to deployed website + screenshots of live site + link to GitHub repo.

Analysis / Evaluation

What worked, limitations, future improvements.

Conclusion

Appendix (migrations listing, useful commands, sample environment file structure)

E) Screenshots checklist (what to capture)

Login page

Registration page

User profile (before/after editing)

Event listing (with search or filter in action)

Event create/edit form (validation)

Event show page (image, description, comments)

Ticket booking flow (confirmation)

Admin dashboard summary page

GitHub repo root + README screenshot
Take images at 1280×720 or full-screen; label them in report.

F) Example “Work percentage and contributions” template
Muhammad Umar Rusjanto — 100%
- Designed database schema
- Implemented Event and Category CRUD (models, migrations, controllers)
- Wrote Event views (Blade templates)
- Performed deployment and created README
- Implemented authentication & profile (login/register)
- Implemented Ticket booking and Comment features
- Wrote tests and seeders


G) Mapping deliverables → grading rubric

Design [20 pts]: ER diagram, UI mockups, navigation flow. (Include diagrams and reasoning.)

Implementation [50 pts]: Demonstrate working CRUDs, auth, foreign keys, tests, file uploads. Use screenshots + GitHub code links.

Analysis/evaluation [25 pts]: Write what worked, constraints, bugs, improvements, performance/security notes.

Conclusion [5 pts]: Short summary and lessons learned.

Make the report match these sections exactly so graders can match points.

H) Quick checklist so you don’t forget

 At least 8 features + auth implemented

 At least one FK in DB (e.g., events.category_id)

 Migrations present and runnable

 Seeders + sample accounts

 Tests (basic feature tests)

 Report PDF with ER diagram, screenshots, contributions

 Source pushed to GitHub

 Site deployed and link included in report

I) Minimal example code snippets to copy-paste

TicketController@store (simplified)

    public function store(Request $req, Event $event) {
        $req->validate(['qty'=>'required|integer|min:1']);
        $sold = $event->tickets()->sum('qty');
        if ($sold + $req->qty > $event->capacity) {
            return back()->withErrors(['capacity'=>'Not enough seats available']);
        }
        $ticket = $event->tickets()->create([
          'user_id'=>auth()->id(),
          'qty'=>$req->qty,
          'status'=>'booked',
          'purchased_at'=>now(),
        ]);
        return redirect()->route('tickets.show',$ticket)->with('success','Ticket booked');
    }
