# Pandas_Learnings
All Pandas Dataframe Learnings to Avoid StackOverflow Searches

# Comparing previous row values in Pandas DataFrame

You need eq with shift:

## df['match'] = df.col1.eq(df.col1.shift())
## print (df)
   col1  match
0     1  False
1     3  False
2     3   True
3     1  False
4     2  False
5     3  False
6     2  False
7     2   True
Or instead eq use ==, but it is a bit slowier in large DataFrame:

## df['match'] = df.col1 == df.col1.shift()
## print (df)
   col1  match
0     1  False
1     3  False
2     3   True
3     1  False
4     2  False
5     3  False
6     2  False
7     2   True

# Pandas Rename Column
gapminder.rename(columns={'pop':'population',
                          'lifeExp':'life_exp',
                          'gdpPercap':'gdp_per_cap'}, 
                 inplace=True)
                 
# Pandas Rename Rows
gapminder.rename(index={0:'zero',1:'one'}, inplace=True)

# Pandas Go over each record
for row in df.itertuples(index=True, name='Pandas'):
    print getattr(row, "c1"), getattr(row, "c2")
    
# Dataframe Sort

DataFrame.sort_values(by, axis=0, ascending=True, inplace=False, kind='quicksort', na_position='last')

# Type Casting/ Conversion
\
You have three main options for converting types in pandas:

to_numeric() - provides functionality to safely convert non-numeric types (e.g. strings) to a suitable numeric type. (See also to_datetime() and to_timedelta().)

astype() - convert (almost) any type to (almost) any other type (even if it's not necessarily sensible to do so). Also allows you to convert to categorial types (very useful).

infer_objects() - a utility method to convert object columns holding Python objects to a pandas type if possible.

Read on for more detailed explanations and usage of each of these methods.

1. to_numeric()
The best way to convert one or more columns of a DataFrame to numeric values is to use pandas.to_numeric().

This function will try to change non-numeric objects (such as strings) into integers or floating point numbers as appropriate.

Basic usage
The input to to_numeric() is a Series or a single column of a DataFrame.

>>> s = pd.Series(["8", 6, "7.5", 3, "0.9"]) # mixed string and numeric values
>>> s
0      8
1      6
2    7.5
3      3
4    0.9
dtype: object

>>> pd.to_numeric(s) # convert everything to float values
0    8.0
1    6.0
2    7.5
3    3.0
4    0.9
dtype: float64
As you can see, a new Series is returned. Remember to assign this output to a variable or column name to continue using it:

# convert Series
my_series = pd.to_numeric(my_series)

# convert column "a" of a DataFrame
df["a"] = pd.to_numeric(df["a"])
You can also use it to convert multiple columns of a DataFrame via the apply() method:

# convert all columns of DataFrame
df = df.apply(pd.to_numeric) # convert all columns of DataFrame

# convert just columns "a" and "b"
df[["a", "b"]] = df[["a", "b"]].apply(pd.to_numeric)
As long as your values can all be converted, that's probably all you need.

Error handling
But what if some values can't be converted to a numeric type?

to_numeric() also takes an errors keyword argument that allows you to force non-numeric values to be NaN, or simply ignore columns containing these values.

Here's an example using a Series of strings s which has the object dtype:

>>> s = pd.Series(['1', '2', '4.7', 'pandas', '10'])
>>> s
0         1
1         2
2       4.7
3    pandas
4        10
dtype: object
The default behaviour is to raise if it can't convert a value. In this case, it can't cope with the string 'pandas':

>>> pd.to_numeric(s) # or pd.to_numeric(s, errors='raise')
ValueError: Unable to parse string
Rather than fail, we might want 'pandas' to be considered a missing/bad numeric value. We can coerce invalid values to NaN as follows using the errors keyword argument:

>>> pd.to_numeric(s, errors='coerce')
0     1.0
1     2.0
2     4.7
3     NaN
4    10.0
dtype: float64
The third option for errors is just to ignore the operation if an invalid value is encountered:

>>> pd.to_numeric(s, errors='ignore')
# the original Series is returned untouched
This last option is particularly useful when you want to convert your entire DataFrame, but don't not know which of our columns can be converted reliably to a numeric type. In that case just write:

df.apply(pd.to_numeric, errors='ignore')
The function will be applied to each column of the DataFrame. Columns that can be converted to a numeric type will be converted, while columns that cannot (e.g. they contain non-digit strings or dates) will be left alone.

Downcasting
By default, conversion with to_numeric() will give you either a int64 or float64 dtype (or whatever integer width is native to your platform).

That's usually what you want, but what if you wanted to save some memory and use a more compact dtype, like float32, or int8?

to_numeric() gives you the option to downcast to either 'integer', 'signed', 'unsigned', 'float'. Here's an example for a simple series s of integer type:

>>> s = pd.Series([1, 2, -7])
>>> s
0    1
1    2
2   -7
dtype: int64
Downcasting to 'integer' uses the smallest possible integer that can hold the values:

>>> pd.to_numeric(s, downcast='integer')
0    1
1    2
2   -7
dtype: int8
Downcasting to 'float' similarly picks a smaller than normal floating type:

>>> pd.to_numeric(s, downcast='float')
0    1.0
1    2.0
2   -7.0
dtype: float32
2. astype()
The astype() method enables you to be explicit about the dtype you want your DataFrame or Series to have. It's very versatile in that you can try and go from one type to the any other.

Basic usage
Just pick a type: you can use a NumPy dtype (e.g. np.int16), some Python types (e.g. bool), or pandas-specific types (like the categorical dtype).

Call the method on the object you want to convert and astype() will try and convert it for you:

# convert all DataFrame columns to the int64 dtype
df = df.astype(int)

# convert column "a" to int64 dtype and "b" to complex type
df = df.astype({"a": int, "b": complex})

# convert Series to float16 type
s = s.astype(np.float16)

# convert Series to Python strings
s = s.astype(str)

# convert Series to categorical type - see docs for more details
s = s.astype('category')
Notice I said "try" - if astype() does not know how to convert a value in the Series or DataFrame, it will raise an error. For example if you have a NaN or inf value you'll get an error trying to convert it to an integer.

As of pandas 0.20.0, this error can be suppressed by passing errors='ignore'. Your original object will be return untouched.

Be careful
astype() is powerful, but it will sometimes convert values "incorrectly". For example:

>>> s = pd.Series([1, 2, -7])
>>> s
0    1
1    2
2   -7
dtype: int64
These are small integers, so how about converting to an unsigned 8-bit type to save memory?

>>> s.astype(np.uint8)
0      1
1      2
2    249
dtype: uint8
The conversion worked, but the -7 was wrapped round to become 249 (i.e. 28 - 7)!

Trying to downcast using pd.to_numeric(s, downcast='unsigned') instead could help prevent this error.

3. infer_objects()
Version 0.21.0 of pandas introduced the method infer_objects() for converting columns of a DataFrame that have an object datatype to a more specific type (soft conversions).

For example, here's a DataFrame with two columns of object type. One holds actual integers and the other holds strings representing integers:

>>> df = pd.DataFrame({'a': [7, 1, 5], 'b': ['3','2','1']}, dtype='object')
>>> df.dtypes
a    object
b    object
dtype: object
Using infer_objects(), you can change the type of column 'a' to int64:

>>> df = df.infer_objects()
>>> df.dtypes
a     int64
b    object
dtype: object
Column 'b' has been left alone since its values were strings, not integers. If you wanted to try and force the conversion of both columns to an integer type, you could use df.astype(int) instead.

# So I want the \n to actually be a new line, and not just to show me the "\n" string.

9

If you're running this in the Python interpreter, it is the regular behavior of the interpreter to show newlines as "\n" instead of actual newlines, because it makes it easier to debug the output. If you want to get actual newlines within the interpreter, you should print the string you get.

If this is what the program is outputting (i.e.: You're getting newline escape sequences from the external program), you should use the following:

OUTPUT = stdout.read()
formatted_output = OUTPUT.replace('\\n', '\n')
print formatted_output
This will replace escaped newlines by actual newlines in the output string.
