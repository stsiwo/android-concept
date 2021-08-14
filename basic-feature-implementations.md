# Basic Feature Implementations

- [X] add an event lister to a UI element

   steps) use _findViewById_ and get the target element and add _setOnXXXListener_ with its handler. 

   _How to organize event listenrs? should I put in UI components or ViewModel?_ Should be inside UI components since event listeners is a UI concern. 
   
   ref: https://medium.com/@CodyEngel/4-ways-to-implement-onclicklistener-on-android-9b956cbd2928
   ref (kotlin): https://stackoverflow.com/questions/44301301/android-how-to-achieve-setonclicklistener-in-kotlin
   ref (best way): create an event handler at an activity and put the name on _onClick_ attribute on the layout https://developer.android.com/training/basics/firstapp/starting-activity
   

- [X] create a link to another activity
- [ ] Navigation
- [ ] Deep Link
- [X] Open another app
- [X] LiveData
- [X] TextWatcher

   ref: https://stackoverflow.com/questions/40569436/kotlin-addtextchangelistener-lambda

- [ ] Intent and Intent Filter
- [X] ConstraintLayout (Responsive UI)
- [ ] MotionLayout (add motion to your layout)
- [ ] RecyclerView
- [ ] CardView (similar layout for list items)
- [ ] SlidingPaneLayout (Two Page Layout & Responsive)
- [ ] <include>/<merge> (reuse layouts)
- [ ] ViewStub (deferring/lazy loading layout: load the layout only when needed)
- [ ] LinearLayout (make all children layout linear e.g., horizontally/vertically)
- [ ] AdapterView (ViewGroup that displays items loaded into an adapter)
- [ ] RelativeLayout (display child views in relative positions. ConstraintLayout is preferrable over this since performance and flexibility)
- [ ] Custom View Component (if you have time. i think this is an advanced topic)
- [ ] foldable (if you have time. i think this is an advanced topic)
- [ ] style and theme
- [ ] dark theme
- [ ] Adaptive icons
- [ ] floating Action Button (a floating button at right bottom corner)
- [ ] Clip Views (chage the shape of a veiw)
- [ ] TextView (autosizing, downloadable font, ...)
- [ ] Button
- [ ] Checkboxes 
- [ ] RadioButton
- [ ] Toggle Buttons 
- [ ] Spinner
- [ ] Pckers
- [ ] Tooltips
- [ ] Notification 
- [ ] Conversations (a list of chat freinds when swip downward from the top)
- [ ] Bubble (floating circle icon which can be expanded to start conversation)
- [ ] app bar (IMPORTANT: your top menu bar) 
- [ ] system bars (the top and the bottom black bar)
- [ ] Swipe

