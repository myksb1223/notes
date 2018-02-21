## CoordinatorLayout with RecyclerView Collapse

I designed a layout so that `CollapsingToolbarLayout` is collapsed when `RecyclerView` is scrolled.

But there is one issue makes it hard for me.

RecyclerView's content height doesn't work properly. So, last item is cut off about navigation bar size.

I struggled to solve it... Maybe spent 6hours...

Solution is very simple... just delay function calls.

```java
new Handler().postDelayed(new Runnable() {
  @Override
  public void run() {
    mAdapter.notifyDataSetChanged();
  }
}, 1000);

```
