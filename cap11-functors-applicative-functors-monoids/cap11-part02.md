Functors redux
==============

Já falamos sobre functors no próprio <a href="making-our-own-types-and-typeclasses#the-functor-typeclass"> capítulo deles</a>. Se você ainda não leu ele, provavelmente deveria dar uma olhada agora, ou talvez 
depois quando você tiver um pouco mais de tempo. Ou você pode simplemente fingir que já leu ele.

Ainda assim, aqui vai uma rápida revisão: Functors são coisas mapeaveis, assim como listas, [code]Maybe[/code]s, árvores, e tal. Em Haskell, eles são descritos pela typeclass [code]Functor[/code], que tem apenas um método de typeclass, chamado [code]fmap[/code], que tem o tipo [code]fmap :: (a -&gt; b) -&gt; f a -&gt; f b[/code].
Ele diz: me de uma função que recebe um [code]a[/code] e que retorna um [code]b[/code] e uma caixa
com um [code]a[/code] (ou um monte deles) dentro dela e eu te darei uma caixa com ou [code]b[/code] 
(ou um monte deles) dentro dela.

<em>Um conselho de amigo.</em> Muitas vezes a analogia das caixas é usada para nos ajudar a ter 
alguma intuição sobre como functors funcionam, e depois, provavelmente vamos usar a mesma analogia
para applicative functors e monads. É uma analogia bacana que ajuda as pessoas a entenderem functors
em um primeiro momento, apenas não leve isso tão ao pé da letra, porque para algumas functors essa 
analogia da caixa não se aplica muito bem. Um termo mais correto para definir o que as functors 
realmente são seria <i>contexto computacional</i>. O contexto pode ser que a computação pode ter 
um valor ou pode ter uma falha ([code]Maybe[/code] e [code]Either a[/code]) ou ele pode ter 
mais valores (listas), coisas desse tipo.

Se quisermos fazer um tipo construtor uma instância de [code]Functor[/code], ele deverá ter um tipo 
de [code]* -&gt; *[/code], o que significa que ele recebe exatamente um tipo concreto como 
um tipo de parâmetro. Por exemplo, de um [code]Maybe[/code] pode ser feita uma instância porque ele recebe um tipo como parâmetro para produzir um tipo concreto, como [code]Maybe Int[/code] ou 
[code]Maybe String[/code]. Se um tipo construtor tem dois parâmetros, como [code]Either[/code], temos 
de aplicar parcialmente o tipo construtor até que ele só tenha um parâmetro do tipo. Portanto, 
não podemos escrever [code]instance Functor Either where[/code], mas podemos escrever 
[code]instance Functor (Either a) where[/code] e, em seguida, se imaginarmos que [code]fmap[/code] 
é só para [code]Either a[/code], ele teria uma declaração de tipo de [code]fmap :: (b -&gt; c) -&gt; Either a b -&gt; Either a c[/code]. Como você pode ver, a parte [code]Either a[/code] é fixa, porque [code]Either a[/code] recebe apenas um tipo como parâmetro, ao passo que se [code]Either[/code] recebesse dois então [code]fmap :: (b -&gt; c) -&gt; Either b -&gt; Either c[/code] não iria fazer sentido.

Aprendemos por enquanto como que um monte de tipos (bem, tipos construtores na verdade) são 
instâncias de [code]Functor[/code], como [code][][/code], [code]Maybe[/code], [code]Either a[/code] 
e o tipo [code]Tree[/code] que nós mesmo fizemos. Dizemos como queremos mapear funções para o nosso próprio bem.
Nesse capítulo, vamos dar uma olhada em mais duas instâncias de functor, chamadas de [code]IO[/code] e 
[code](-&gt;) r[/code].

Se algum valor tiver o tipo de, digamos, [code]IO String[/code], isso irá significar que uma ação I/O,
quando executada, irá até o mundo real e trazer alguma string para nós, que será o nosso resultado. 
Podemos usar [code]&lt;-[/code] na sintaxe <i>do</i> para atrelar esse resultado a um nome.  Já 
mencionamos que ações I/O são como pequenas caixas com perninhas que vão até o mundo real e pegam
algum valor para nós. Nós podemos inspecionar o que elas pegaram, mas depois de inspecionar, 
nós temos que devolver o valor de volta ao [code]IO[/code]. Ao pensar nessa analogia da caixa com 
pequenas perninhas, nós conseguimos ver como [code]IO[/code] age como um functor.

Vamos ver como [code]IO[/code] é uma instância de [code]Functor[/code]. Quando nós [code]fmap[/code]  uma função sobre uma ação I/O, nós esperamos receber de volta uma ação I/O que faz a mesma coisa, mas que tem a nossa função aplicada sobre seus valores resultantes.


O resultado de um mapping sobre uma ação de I/O será uma ação de I/O, então logo de cara usamos a sintaxe para colar duas ações para fazer uma nova.


The result of mapping something over an I/O action will be an I/O action, so right off the bat we 
use <i>do</i> syntax to glue two actions and make a new one. In the implementation for 
[code]fmap[/code], we make a new I/O action that first performs the original I/O action and 
calls its result [code]result[/code]. Then, we do [code]return (f result)[/code]. 
[code]return[/code] is, as you know, a function that makes an I/O action that doesn't 
do anything but only presents something as its result. The action that a <i>do</i> block produces 
will always have the result value of its last action. That's why we use return to make an I/O 
action that doesn't really do anything, it just presents [code]f result[/code] as the result of 
the new I/O action.


We can play around with it to gain some intuition. It's pretty simple really. Check out this 
piece of code:


The user is prompted for a line and we give it back to the user, only reversed. Here's how to 
rewrite this by using [code]fmap[/code]:


Just like when we [code]fmap[/code] [code]reverse[/code] over [code]Just "blah"[/code] to get 
[code]Just "halb"[/code], we can [code]fmap[/code] [code]reverse[/code] over [code]getLine[/code]. 
[code]getLine[/code] is an I/O action that has a type of [code]IO String[/code] and mapping 
[code]reverse[/code] over it gives us an I/O action that will go out into the real world and 
get a line and then apply [code]reverse[/code] to its result. Like we can apply a function to 
something that's inside a [code]Maybe[/code] box, we can apply a function to what's inside an 
[code]IO[/code] box, only it has to go out into the real world to get something. Then when we 
bind it to a name by using [code]&lt;-[/code], the name will reflect the result that already 
has [code]reverse[/code] applied to it.

The I/O action [code]fmap (++"!") getLine[/code] behaves just like [code]getLine[/code], only that 
its result always has [code]"!"[/code] appended to it!

If we look at what [code]fmap[/code]'s type would be if it were limited to [code]IO[/code], 
it would be [code]fmap :: (a -&gt; b) -&gt; IO a -&gt; IO b[/code]. [code]fmap[/code] takes a 
function and an I/O action and returns a new I/O action that's like the old one, except that the 
function is applied to its contained result.

If you ever find yourself binding the result of an I/O action to a name, only to apply a 
function to that and call that something else, consider using [code]fmap[/code], because it 
looks prettier. If you want to apply multiple transformations to some data inside a functor, 
you can declare your own function at the top level, make a lambda function or ideally, use 
function composition:


As you probably know, [code]intersperse '-' . reverse . map toUpper[/code] is a function 
that takes a string, maps [code]toUpper[/code] over it, the applies [code]reverse[/code] 
to that result and then applies [code]intersperse '-'[/code] to that result. It's like writing 
[code](\xs -&gt; intersperse '-' (reverse (map toUpper xs)))[/code], only prettier.

Another instance of [code]Functor[/code] that we've been dealing with all along but didn't 
know was a [code]Functor[/code] is [code](-&gt;) r[/code]. You're probably slightly confused now, 
since what the heck does [code](-&gt;) r[/code] mean? The function type [code]r -&gt; a[/code] 
can be rewritten as [code](-&gt;) r a[/code], much like we can write [code]2 + 3[/code] as 
[code](+) 2 3[/code]. When we look at it as [code](-&gt;) r a[/code], we can see  [code](-&gt;)[/code] 
in a slighty different light, because we see that it's just a type constructor that takes two 
type parameters, just like [code]Either[/code]. But remember, we said that a type constructor 
has to take exactly one type parameter so that it can be made an instance of [code]Functor[/code]. 
That's why we can't make [code](-&gt;)[/code] an instance of [code]Functor[/code], but if we 
partially apply it to [code](-&gt;) r[/code], it doesn't pose any problems. If the syntax allowed 
for type constructors to be partially applied with sections (like we can partially apply 
	[code]+[/code] by doing [code](2+)[/code], which is the same as [code](+) 2[/code]), 
you could write [code](-&gt;) r[/code] as [code](r -&gt;)[/code]. How are functions functors? 
Well, let's take a look at the implementation, which lies in [code]Control.Monad.Instances[/code]

We usually mark functions that take anything and return anything as [code]a -&gt; b[/code]. 
[code]r -&gt; a[/code] is the same thing, we just used different letters for the type variables.

If the syntax allowed for it, it could have been written as


But it doesn't, so we have to write it in the former fashion.

First of all, let's think about [code]fmap[/code]'s type. 
It's [code]fmap :: (a -&gt; b) -&gt; f a -&gt; f b[/code]. Now what we'll do is mentally replace all 
the [code]f[/code]'s, which are the role that our functor instance plays, with [code](-&gt;) r[/code]'s. 
We'll do that to see how [code]fmap[/code] should behave for this particular instance. 
We get [code]fmap :: (a -&gt; b) -&gt; ((-&gt;) r a) -&gt; ((-&gt;) r b)[/code]. 
Now what we can do is write the [code](-&gt;) r a[/code] and [code](-&gt; r b)[/code] 
types as infix [code]r -&gt; a[/code] and [code]r -&gt; b[/code], like we normally do with functions. 
What we get now is [code]fmap :: (a -&gt; b) -&gt; (r -&gt; a) -&gt; (r -&gt; b)[/code].

Hmmm OK. Mapping one function over a function has to produce a function, just like mapping a 
function over a [code]Maybe[/code] has to produce a [code]Maybe[/code] and mapping a function over a 
list has to produce a list. What does the type 
[code]fmap :: (a -&gt; b) -&gt; (r -&gt; a) -&gt; (r -&gt; b)[/code] for this instance tell us? 
Well, we see that it takes a function from [code]a[/code] to [code]b[/code] and a function 
from [code]r[/code] to [code]a[/code] and returns a function from [code]r[/code] to [code]b[/code]. 
Does this remind you of anything? Yes! Function composition! We pipe the output of 
[code]r -&gt; a[/code] into the input of [code]a -&gt; b[/code] to get a function 
[code]r -&gt; b[/code], which is exactly what function composition is about. If you look at how 
the instance is defined above, you'll see that it's just function composition. Another way to 
write this instance would be:

This makes the revelation that using [code]fmap[/code] over functions is just composition sort of 
obvious. Do [code]:m + Control.Monad.Instances[/code], since that's where the instance is defined 
and then try playing with mapping over functions.

We can call [code]fmap[/code] as an infix function so that the resemblance to [code].[/code] is clear. 
In the second input line, we're mapping [code](*3)[/code] over [code](+100)[/code], which results in a 
function that will take an input, call [code](+100)[/code] on that and then call [code](*3)[/code] 
on that result. We call that function with [code]1[/code].

How does the box analogy hold here? Well, if you stretch it, it holds. When we use 
[code]fmap (+3)[/code] over [code]Just 3[/code], it's easy to imagine the [code]Maybe[/code] as a 
box that has some contents on which we apply the function [code](+3)[/code]. But what about when 
we're doing [code]fmap (*3) (+100)[/code]? Well, you can think of the function [code](+100)[/code] 
as a box that contains its eventual result. Sort of like how an I/O action can be thought of as a 
box that will go out into the real world and fetch some result. Using [code]fmap (*3)[/code] on 
[code](+100)[/code] will create another function that acts like [code](+100)[/code], only before 
producing a result, [code](*3)[/code] will be applied to that result. Now we can see how 
[code]fmap[/code] acts just like [code].[/code] for functions.

The fact that [code]fmap[/code] is function composition when used on functions isn't so terribly 
useful right now, but at least it's very interesting. It also bends our minds a bit and let us see 
how things that act more like computations than boxes ([code]IO[/code] and [code](-&gt;) r[/code]) 
can be functors. The function being mapped over a computation results in the same computation but 
the result of that computation is modified with the function.

Before we go on to the rules that [code]fmap[/code] should follow, let's think about the type of 
[code]fmap[/code] once more. Its type is [code]fmap :: (a -&gt; b) -&gt; f a -&gt; f b[/code]. 
We're missing the class constraint [code](Functor f) =&gt;[/code], but we left it out here for brevity, 
because we're talking about functors anyway so we know what the [code]f[/code] stands for. 
When we first learned about <a href="higher-order-functions#curried-functions">curried functions</a>, 
we said that all Haskell functions actually take one parameter. A function 
[code]a -&gt; b -&gt; c[/code] actually takes just one parameter of type [code]a[/code] and then 
returns a function [code]b -&gt; c[/code], which takes one parameter and returns a [code]c[/code]. 
That's how if we call a function with too few parameters (i.e. partially apply it), 
we get back a function that takes the number of parameters that we left out (if we're thinking about 
	functions as taking several parameters again). So [code]a -&gt; b -&gt; c[/code] can be written 
as [code]a -&gt; (b -&gt; c)[/code], to make the currying more apparent.


In the same vein, if we write [code]fmap :: (a -&gt; b) -&gt; (f a -&gt; f b)[/code], we can think 
of [code]fmap[/code] not as a function that takes one function and a functor and returns a functor, 
but as a function that takes a function and returns a new function that's just like the old one, 
only it takes a functor as a parameter and returns a functor as the result. It takes an 
[code]a -&gt; b[/code] function and returns a function [code]f a -&gt; f b[/code]. This is called 
<i>lifting</i> a function. Let's play around with that idea by using GHCI's [code]:t[/code] command:


The expression [code]fmap (*2)[/code] is a function that takes a functor [code]f[/code] over numbers 
and returns a functor over numbers. That functor can be a list, a [code]Maybe [/code], an 
[code]Either String[/code], whatever. The expression [code]fmap (replicate 3)[/code] will take a 
functor over any type and return a functor over a list of elements of that type.

When we say <i>a functor over numbers</i>, you can think of that as <i>a functor that has numbers 
in it</i>. The former is a bit fancier and more technically correct, but the latter is usually easier 
to get.

This is even more apparent if we partially apply, say, [code]fmap (++"!")[/code] and then bind it to 
a name in GHCI.
You can think of [code]fmap[/code] as either a function that takes a function and a functor and then 
maps that function over the functor, or you can think of it as a function that takes a function and 
lifts that function so that it operates on functors. Both views are correct and in Haskell, equivalent.
The type [code]fmap (replicate 3) :: (Functor f) =&gt; f a -&gt; f [a][/code] means that the function 
will work on any functor. What exactly it will do depends on which functor we use it on. 
If we use [code]fmap (replicate 3)[/code] on a list, the list's implementation for [code]fmap[/code] 
will be chosen, which is just [code]map[/code]. If we use it on a [code]Maybe a[/code], 
it'll apply [code]replicate 3[/code] to the value inside the [code]Just[/code], or if it's 
[code]Nothing[/code], then it stays [code]Nothing[/code].

Next up, we're going to look at the <em>functor laws</em>. In order for something to be a functor, 
it should satisfy some laws. All functors are expected to exhibit certain kinds of functor-like 
properties and behaviors. They should reliably behave as things that can be mapped over. 
Calling [code]fmap[/code] on a functor should just map a function over the functor, nothing more. 
This behavior is described in the functor laws. There are two of them that all instances of 
[code]Functor[/code] should abide by. They aren't enforced by Haskell automatically, so you have 
to test them out yourself.

<em>The first functor law states that if we map the [code]id[/code] function over a functor, 
the functor that we get back should be the same as the original functor.</em> If we write that a bit 
more formally, it means that [law]fmap id = id[/code]. So essentially, this says that if we do 
[code]fmap id[/code] over a functor, it should be the same as just calling [code]id[/code] on the 
functor. Remember, [code]id[/code] is the identity function, which just returns its parameter unmodified. 
It can also be written as [code]\x -&gt; x[/code]. If we view the functor as something that can be 
mapped over, the [law]fmap id = id[/code] law seems kind of trivial or obvious.

Let's see if this law holds for a few values of functors.


If we look at the implementation of [code]fmap[/code] for, say, [code]Maybe[/code], we can figure out 
why the first functor law holds.

We imagine that [code]id[/code] plays the role of the [code]f[/code] parameter in the implementation. 
We see that if wee [code]fmap id[/code] over [code]Just x[/code], the result will be 
[code]Just (id x)[/code], and because [code]id[/code] just returns its parameter, we can deduce that 
[code]Just (id x)[/code] equals [code]Just x[/code]. So now we know that if we map [code]id[/code] 
over a [code]Maybe[/code] value with a [code]Just[/code] value constructor, we get that same value back.

Seeing that mapping [code]id[/code] over a [code]Nothing[/code] value returns the same value is trivial. 
So from these two equations in the implementation for [code]fmap[/code], we see that the law 
[code]fmap id = id[/code] holds.

<em>The second law says that composing two functions and then mapping the resulting function over a 
functor should be the same as first mapping one function over the functor and then mapping the other 
one.</em> Formally written, that means that [law]fmap (f . g) = fmap f . fmap g[/code]. 
Or to write it in another way, for any functor <i>F</i>, the following should hold: 
[law]fmap (f . g) F = fmap f (fmap g F)[/code].

If we can show that some type obeys both functor laws, we can rely on it having the same fundamental 
behaviors as other functors when it comes to mapping. We can know that when we use [code]fmap[/code] 
on it, there won't be anything other than mapping going on behind the scenes and that it will 
act like a thing that can be mapped over, i.e. a functor. You figure out how the second law holds 
for some type by looking at the implementation of [code]fmap[/code] for that type and then using 
the method that we used to check if [code]Maybe[/code] obeys the first law.

If you want, we can check out how the second functor law holds for [code]Maybe[/code]. 
If we do [code]fmap (f . g)[/code] over [code]Nothing[/code], we get [code]Nothing[/code], 
because doing a [code]fmap[/code] with any function over [code]Nothing[/code] returns 
[code]Nothing[/code]. If we do [code]fmap f (fmap g Nothing)[/code], we get [code]Nothing[/code], 
for the same reason. OK, seeing how the second law holds for [code]Maybe[/code] if it's a 
[code]Nothing[/code] value is pretty easy, almost trivial. How about if it's a 
[code]Just <i>something</i>[/code] value? Well, if we do [code]fmap (f . g) (Just x)[/code], 
we see from the implementation that it's implemented as [code]Just ((f . g) x)[/code], 
which is, of course, [code]Just (f (g x))[/code]. If we do [code]fmap f (fmap g (Just x))[/code], 
we see from the implementation that [code]fmap g (Just x)[/code] is [code]Just (g x)[/code]. 
Ergo, [code]fmap f (fmap g (Just x))[/code] equals [code]fmap f (Just (g x))[/code] and from the 
implementation we see that this equals [code]Just (f (g x))[/code].

If you're a bit confused by this proof, don't worry. Be sure that you understand how 
<a href="higher-order-functions#composition">function composition</a> works. Many times, you can 
intuitively see how these laws hold because the types act like containers or functions. You can 
also just try them on a bunch of different values of a type and be able to say with some certainty 
that a type does indeed obey the laws.

Let's take a look at a pathological example of a type constructor being an instance of the 
[code]Functor[/code] typeclass but not really being a functor, because it doesn't satisfy the laws. 
Let's say that we have a type:

The C here stands for <i>counter</i>. It's a data type that looks much like [code]Maybe a[/code], 
only the [code]Just[/code] part holds two fields instead of one. The first field in the 
[code]CJust[/code] value constructor will always have a type of [code]Int[/code], and it will be some 
sort of counter and the second field is of type [code]a[/code], which comes from the type parameter 
and its type will, of course, depend on the concrete type that we choose for [code]CMaybe a[/code]. 
Let's play with our new type to get some intuition for it.

If we use the [code]CNothing[/code] constructor, there are no fields, and if we use the 
[code]CJust[/code] constructor, the first field is an integer and the second field can be any type. 
Let's make this an instance of [code]Functor[/code] so that everytime we use [code]fmap[/code], 
the function gets applied to the second field, whereas the first field gets increased by 1.

This is kind of like the instance implementation for [code]Maybe[/code], except that when we do 
[code]fmap[/code] over a value that doesn't represent an empty box (a [code]CJust[/code] value), 
we don't just apply the function to the contents, we also increase the counter by 1. 
Everything seems cool so far, we can even play with this a bit:

Does this obey the functor laws? In order to see that something doesn't obey a law, it's enough to 
find just one counter-example.


Ah! We know that the first functor law states that if we map [code]id[/code] over a functor, 
it should be the same as just calling [code]id[/code] with the same functor, but as we've seen 
from this example, this is not true for our [code]CMaybe[/code] functor. Even though it's part 
of the [code]Functor[/code] typeclass, it doesn't obey the functor laws and is therefore not a functor. 
If someone used our [code]CMaybe[/code] type as a functor, they would expect it to obey the functor 
laws like a good functor. But [code]CMaybe[/code] fails at being a functor even though it pretends 
to be one, so using it as a functor might lead to some faulty code. When we use a functor, it 
shouldn't matter if we first compose a few functions and then map them over the functor or if we 
just map each function over a functor in succession. But with [code]CMaybe[/code], it matters, 
because it keeps track of how many times it's been mapped over. Not cool! If we wanted 
[code]CMaybe[/code] to obey the functor laws, we'd have to make it so that the [code]Int[/code] 
field stays the same when we use [code]fmap[/code].

At first, the functor laws might seem a bit confusing and unnecessary, but then we see that if we 
know that a type obeys both laws, we can make certain assumptions about how it will act. If a 
type obeys the functor laws, we know that calling [code]fmap[/code] on a value of that type will 
only map the function over it, nothing more. This leads to code that is more abstract and extensible, 
because we can use laws to reason about behaviors that any functor should have and make functions 
that operate reliably on any functor.

All the [code]Functor[/code] instances in the standard library obey these laws, but you can check 
for yourself if you don't believe me. And the next time you make a type an instance of 
[code]Functor[/code], take a minute to make sure that it obeys the functor laws. Once you've dealt 
with enough functors, you kind of intuitively see the properties and behaviors that they have in 
common and it's not hard to intuitively see if a type obeys the functor laws. But even without the 
intuition, you can always just go over the implementation line by line and see if the laws hold or 
try to find a counter-example.

We can also look at functors as things that output values in a context. For instance, 
[code]Just 3[/code] outputs the value [code]3[/code] in the context that it might or not output 
any values at all. [code][1,2,3][/code] outputs three values—[code]1[/code], [code]2[/code], and 
[code]3[/code], the context is that there may be multiple values or no values. The function 
[code](+3)[/code] will output a value, depending on which parameter it is given.

If you think of functors as things that output values, you can think of mapping over functors as 
attaching a transformation to the output of the functor that changes the value. When we do 
[code]fmap (+3) [1,2,3][/code], we attach the transformation [code](+3)[/code] to the output of 
[code][1,2,3][/code], so whenever we look at a number that the list outputs, [code](+3)[/code] 
will be applied to it. Another example is mapping over functions. When we do 
[code]fmap (+3) (*3)[/code], we attach the transformation [code](+3)[/code] to the eventual output 
of [code](*3)[/code]. Looking at it this way gives us some intuition as to why using 
[code]fmap[/code] on functions is just composition ([code]fmap (+3) (*3)[/code] equals 
	[code](+3) . (*3)[/code], which equals [code]\x -&gt; ((x*3)+3)[/code]), because we take a 
function like [code](*3)[/code] then we attach the transformation [code](+3)[/code] to its output. 
The result is still a function, only when we give it a number, it will be multiplied by three and 
then it will go through the attached transformation where it will be added to three. 
This is what happens with composition.
