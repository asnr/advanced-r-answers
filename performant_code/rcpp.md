Getting started
---------------

 1. Figure out what the corresponding base R function is:

    ```cpp
    double f1(NumericVector x) {
      int n = x.size();
      double y = 0;

      for(int i = 0; i < n; ++i) {
        y += x[i] / n;
      }
      return y;
    }
    // f1 = mean

    NumericVector f2(NumericVector x) {
      int n = x.size();
      NumericVector out(n);

      out[0] = x[0];
      for(int i = 1; i < n; ++i) {
        out[i] = out[i - 1] + x[i];
      }
      return out;
    }
    // f2 = cumsum
    

    bool f3(LogicalVector x) {
      int n = x.size();

      for(int i = 0; i < n; ++i) {
        if (x[i]) return true;
      }
      return false;
    }
    // f3 = any

    
    int f4(Function pred, List x) {
      int n = x.size();

      for(int i = 0; i < n; ++i) {
        LogicalVector res = pred(x[i]);
        if (res[0]) return i + 1;
      }
      return 0;
    }
    // f4 = which.max
    

    NumericVector f5(NumericVector x, NumericVector y) {
      int n = std::max(x.size(), y.size());
      NumericVector x1 = rep_len(x, n);
      NumericVector y1 = rep_len(y, n);

      NumericVector out(n);

      for (int i = 0; i < n; ++i) {
        out[i] = std::min(x1[i], y1[i]);
      }

      return out;
    }
    // f5 = pmin
    ```

 2. Rewrite R functions with RCpp.

    ```cpp
    bool my_all(LogicalVector x) {
        int n = x.size();
        for (int i = 0; i < n; i++) {
            if (!x[i]) return false;
        }
        return true;
    }

    NumericVector my_cumprod(NumericVector x) {
        int n = x.size();
        NumericVector out(n);
        if (n >=  1) out[0] = x[0];
        for (int i = 1; i < n; i++) {
            out[i] = x[i]*out[i-1];
        }
        return out;
    }

    NumericVector my_cummin(NumericVector x) {
        int n = x.size();
        NumericVector out(n);
        if (n >= 1) out[0] = x[0];
        for (int i = 1; i < n; i++) {
            out[i] = std::min(x[i], out[i-1]);
        }
        return out;
    }


    NumericVector my_cummax(NumericVector x) {
        int n = x.size();
        NumericVector out(n);
        if (n >= 1) out[0] = x[0];
        for (int i = 1; i < n; i++) {
            out[i] = std::max(x[i], out[i-1]);
        }
        return out;
    }

    NumericVector my_diff(NumericVector x, int lag=1, int differences=1) {
        if (differences == 0)
            return x;

        int n = x.size();
        NumericVector out(n - lag);
        for (int i = lag; i < n; i++) {
            out[i-lag] = x[i] - x[i-lag];
        }
        return my_diff(out, lag, differences - 1);
    }

    // online variance
    double my_var(NumericVector x) {
        double n = 0;
        double mean = 0;
        double M2 = 0;

        int len = x.size();
        for (int i = 0; i < len; i++) {
            n++;
            delta = x[i] - mean;
            mean += delta/n;
            M2 += delta*(x[i] - mean);
        }
        return M2 / (n - 1);
    }
    ```

Missing Values
--------------

 1. Rewrite some R functions, this time dealing with missing values.

    ```cpp
    double my_max(NumericVector x, bool na_rm=true) {
        int n = x.size();
        bool max_set = false;
        double max;
        for (int i = 0; i < n; i++) {
            if (NumericVector::is_na(x[i])) {
                if (!na_rm)
                    return NA_REAL;
                // if (na_rm), then do nothing this iteration    
            } else {
                if (max_set)
                    max = std::max(max, x[i]);
                else
                    max = x[i];
            }
        }
        return max;
    }
    ```


STL
---

 1. Rewrite `median.default()` using `partial_sort`.

    ```cpp
    #include <algorithm>
    #include <Rcpp.h>

    using namespace Rcpp;

    // [[Rcpp::export]]
    double my_median_default(NumericVector x, bool na_rm=false) {
        
        NumericVector x_no_na = x[!is_na(x)];
        if (!na_rm && x_no_na.size() < x.size())
            return NA_REAL;
        
        int half_dist = x_no_na.size()/2;
        NumericVector::iterator middle = x_no_na.begin() + half_dist + 1;
        std::partial_sort(x_no_na.begin(), middle,
                          x_no_na.end());
        
        middle = x_no_na.begin() + half_dist;
        if (x_no_na.size() % 2 == 0) 
            return ( (*middle) + (*(middle-1)) )/2;
        else
            return *middle;
    }
    ```

 2. Rewrite `%in%` using `unordered_set`
    ```
    // [[Rcpp::plugins(cpp11)]]
    #include <Rcpp.h>
    #include <unordered_set>
    using namespace Rcpp;
     
    // Note we cannot implement this trivially for NumericVector because String
    // does not have a hash implementation:
    // undefined reference to `std::hash<Rcpp::String>::operator()(Rcpp::String) const
    // [[Rcpp::export]]
    LogicalVector my_in(NumericVector x, NumericVector table) {
        
        std::unordered_set<double> table_set;
        NumericVector::iterator it;
        for (it = table.begin(); it != table.end(); it++) {
            table_set.insert(*it);
        }
        
        LogicalVector out(x.size());
        LogicalVector::iterator l_it;
        for (l_it = out.begin(), it = x.begin(); l_it != out.end(); l_it++, it++) {
            if (table_set.count(*it) > 0)
                *l_it = true;
            else
                *l_it = false;
        }
        
        return out;
    }
    ```
    
 3. Rewrite `unique()` using `unordered_set`
    ```cpp
    // [[Rcpp::plugins(cpp11)]]
    #include <Rcpp.h>
    #include <unordered_set>
    using namespace Rcpp;

    // [[Rcpp::export]]
    NumericVector my_unique(NumericVector x) {
        return wrap(std::unordered_set<double>(x.begin(), x.end()));
    }
    ```

 4. Rewrite `max()` using `std::max()`
    ```cpp
    // [[Rcpp::plugins(cpp11)]]
    #include <Rcpp.h>
    using namespace Rcpp;

    // [[Rcpp::export]]
    double my_max(NumericVector x, na_rm = false) {
        
        NumericVector x_no_na = x[!is_na(x)];
        if (!na_rm && x_no_na.size() < x.size())
            return NA_REAL;
        
        return *(std::max_element(x_no_na.begin(), x_no_na.end()));
    }
