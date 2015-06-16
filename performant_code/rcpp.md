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