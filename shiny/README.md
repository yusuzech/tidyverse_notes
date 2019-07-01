# Key Notes:

1. Disable scientific notation in your app by writing `options(scipen = 999)`.

  * It is extremely important because when you have filter on IDs, the IDs could be integer, 
  and when integer is converted to string in `selectInput`, it converts number like 100000 to 1e5, and when used in filters, in will results in error!
  * I debugged it for hours in my app and finally found it!
