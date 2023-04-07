A problem I am faced with sometimes is to run a regression with a set of
event-time dummy variables as regressors. The norm is to use event time
-1 as the omitted category so the coefficients show the “effect” of the
treatment in `\tau` periods since the event.

It turns out you can do the creation and inclusion in a regression of
these event time dummies pretty easily in R.

First, use the `fastDummies` package to create the dummies and “merge”
them back on to the original data. The key function is `dummy_cols`
which takes in a column that we want to turn into the dummies. So if the
column is “event\_time” then we will end up with a set of dummy
variables for each value of event time. The other arguments to
`dummy_cols` are a `select_columns` list and an instruction to remove
the selected columns from the result.

    #install.packages("fastDummies") #run if package not installed
    dt <- cbind(dt, #cbind so we can merge the dummies back to the original
                fastDummies::dummy_cols( 
                  dt[,.(event_time)], #the column we want to turn into dummies
                  select_columns = "event_time", #select this column
                  remove_selected_columns = T)) #don't want to have duplicate names

Now that our data.table has a a bunch of new columns that are the dummy
variables, we need to get them into a regression.

The easiest way to do this is to get a new object that is just a vector
of the dummy variables we just created, turn that vector into a formula,
and then include the formula in the regression. The other key here is to
get the names in the order of the event time, so when they go in the
regression we don’t get confused by the output ordering. To do that, add
another vector for the order in which we want the names to appear.

    #get formula for regression so we don't have to type it in when we add more weeks
    names_dt <- data.table(names = names(dt)[grepl("event_time", names(dt))],
                            order = as.numeric(
                              gsub(".*?_","",names(dt)[grepl("event_time",names(dt))]))
                            )
    setorder(names_dt, order)

Now include the names in the regression using the `as.formula()`
function, with some use of the `collapse` feature in the `paste`
function. Notice the use of `paste0` vs `paste`. If you want to include
other variables in the regression, you can just insert them in the
`paste0` as additional arguments.

    #this will show coefficients relative to omitted event time period
    reg <- lm(
      as.formula(
        paste0("outcome ~ ", 
               paste(names_dt$names, collapse = " + ")
        )
      ),
      data = dt)