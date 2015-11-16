
## Excitations, Ribbon operators, Ground states in Quantum Double Models

#### Defining different models - Quantum Double of Z<sub>2</sub>, S<sub>3</sub>, D<sub>4</sub>


```python
QDM_Toric = SymmetricGroup(2)
QDM_Toric
```




    Symmetric group of order 2! as a permutation group




```python
QDM_S3 = SymmetricGroup(3)
QDM_S3
```




    Symmetric group of order 3! as a permutation group




```python
QDM_D4 = DihedralGroup(4)
QDM_D4
```




    Dihedral group of order 8 as a permutation group



* * * *

#### Developing the machinery to compute the number of excitations.

##### 1. Computing the centralizers of the conjugacy class of the group.


```python
def centralizer_conjugacy_class_QDM_generic(QDM_group):
    cent_QDM_group = []
    for conj_class in QDM_group.conjugacy_classes():
        centralizer = QDM_group.centralizer(conj_class.an_element())
        cent_QDM_group.append(centralizer)
    return cent_QDM_group
```


```python
cent_toric = centralizer_conjugacy_class_QDM_generic(QDM_Toric)
cent_toric
```




    [Subgroup of (Symmetric group of order 2! as a permutation group) generated by [(1,2)],
     Subgroup of (Symmetric group of order 2! as a permutation group) generated by [(1,2)]]




```python
cent_s3 = centralizer_conjugacy_class_QDM_generic(QDM_S3)
cent_s3
```




    [Subgroup of (Symmetric group of order 3! as a permutation group) generated by [(2,3), (1,3)],
     Subgroup of (Symmetric group of order 3! as a permutation group) generated by [(1,2)],
     Subgroup of (Symmetric group of order 3! as a permutation group) generated by [(1,2,3)]]




```python
cent_d4 = centralizer_conjugacy_class_QDM_generic(QDM_D4)
cent_d4
```




    [Subgroup of (Dihedral group of order 8 as a permutation group) generated by [(1,2,3,4), (1,4)(2,3)],
     Subgroup of (Dihedral group of order 8 as a permutation group) generated by [(2,4), (1,3)(2,4)],
     Subgroup of (Dihedral group of order 8 as a permutation group) generated by [(1,2)(3,4), (1,3)(2,4)],
     Subgroup of (Dihedral group of order 8 as a permutation group) generated by [(1,2,3,4), (1,3)(2,4)],
     Subgroup of (Dihedral group of order 8 as a permutation group) generated by [(1,2,3,4), (1,4)(2,3)]]



##### 2. The character table gives the number of trace of the irreducible representations (but the trace is used at a later stage).


```python
def character_table_centralizers(centralizers_generic_group):
    char_table = []
    for subgroup in centralizers_generic_group:
        char_table.append(subgroup.character_table())
    return char_table
```


```python
cent_toric_centralizer_character = character_table_centralizers(cent_toric)
cent_toric_centralizer_character
```




    [
    [ 1 -1]  [ 1 -1]
    [ 1  1], [ 1  1]
    ]




```python
cent_s3_centralizer_character_table = character_table_centralizers(cent_s3)
cent_s3_centralizer_character_table
```




    [
    [ 1 -1  1]           [         1          1          1]
    [ 2  0 -1]  [ 1 -1]  [         1      zeta3 -zeta3 - 1]
    [ 1  1  1], [ 1  1], [         1 -zeta3 - 1      zeta3]
    ]




```python
cent_d4_centralizer_character_table = character_table_centralizers(cent_d4)
cent_d4_centralizer_character_table
```




    [
    [ 1  1  1  1  1]                              
    [ 1 -1 -1  1  1]  [ 1  1  1  1]  [ 1  1  1  1]
    [ 1 -1  1 -1  1]  [ 1 -1 -1  1]  [ 1 -1 -1  1]
    [ 1  1 -1 -1  1]  [ 1 -1  1 -1]  [ 1 -1  1 -1]
    [ 2  0  0  0 -2], [ 1  1 -1 -1], [ 1  1 -1 -1],
    
                                   [ 1  1  1  1  1]
    [     1      1      1      1]  [ 1 -1 -1  1  1]
    [     1     -1      1     -1]  [ 1 -1  1 -1  1]
    [     1 -zeta4     -1  zeta4]  [ 1  1 -1 -1  1]
    [     1  zeta4     -1 -zeta4], [ 2  0  0  0 -2]
    ]



##### 3. Computing the number of excitations by counting the number of rows in the character table.


```python
def excitations_count(QDM_group):
    count = 0
    generic_centralizer_character_table = character_table_centralizers(centralizer_conjugacy_class_QDM_generic(QDM_group))
    for char_table in generic_centralizer_character_table:
        count += char_table.nrows()
    return count
```


```python
QDM_toric_excitations = excitations_count(QDM_Toric)
QDM_toric_excitations
```




    4




```python
QDM_S3_excitations = excitations_count(QDM_S3)
QDM_S3_excitations
```




    8




```python
QDM_D4_excitations = excitations_count(QDM_D4)
QDM_D4_excitations
```




    22



* * * 

#### Developing the machinery to compute the excitations condense on a given boundary

##### 1. Computing the character related to the irreducible representation of the group.


```python
def character_excitation(G, conjugacy_class, g, h):
    k_h = 0
    for g_1 in G:
        if h*g_1 == g_1*conjugacy_class.an_element():
            k_h = g_1
            break
    if g*h == h*g and k_h != 0:
        return k_h^-1*g*k_h
    else:
        return 0
```

##### 2. Computing the character related to a particular boundary.


```python
def character_subgroup(G, subgroup, g, h):
    sum = 0
    if h*g == g*h:
        for g_1 in G:
            if g_1*g*g_1^-1 in subgroup and g_1*h*g_1^-1 in subgroup:
                sum = sum + 1
    return sum/len(subgroup)
```

##### 3. Computing the inner product terms of the above characters.


```python
def inner_product_of_characters(QDM_group, subgroup, conjugacy_class):
    inner_product_terms = []
    for g in QDM_group:
        for h in QDM_group:
            if character_subgroup(QDM_group, subgroup, g, h) != 0 and character_excitation(QDM_group, conjugacy_class, g, h) != 0:
                inner_product_terms.append([character_subgroup(QDM_group, subgroup, g, h), character_excitation(QDM_group, conjugacy_class, g, h)])
    return inner_product_terms
```


```python
inner_product_of_characters(QDM_S3, QDM_S3.subgroups()[5], QDM_S3.conjugacy_classes()[0])
```




    [[1, ()], [1, (1,2)], [1, (1,2,3)], [1, (1,3,2)], [1, (2,3)], [1, (1,3)]]



<style TYPE="text/css">
code.has-jax {font: inherit; font-size: 100%; background: inherit; border: inherit;}
</style>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {
        inlineMath: [['$','$'], ['\\(','\\)']],
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'] // removed 'code' entry
    }
});
MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

$1*tr_{\pi_{i}}(e) + 1*tr_{\pi_{i}}(1,2) + 1*tr_{\pi_{i}}(1,2,3) + 1*tr_{\pi_{i}}(1,3,2) + 1*tr_{\pi_{i}}(2,3) + 1*tr_{\pi_{i}}(1,3)$

From the character table for $S_{3}$, and labelling each excitation 

$\begin{matrix}
  \{e\} & \{\tau\} & \{\sigma\} &  & \\
  1 & -1 &  1 &-> & tr_{\pi_{2}} &-> &B\\
  2 &  0 & -1 &-> & tr_{\pi_{3}} &-> &C\\
  1 &  1 &  1 &-> & tr_{\pi_{1}} &-> &A
\end{matrix}$

Therefore $A$ condenses on the boundary as the inner product is greater than zero, the others go to zero. 


```python
inner_product_of_characters(QDM_S3, QDM_S3.subgroups()[5], QDM_S3.conjugacy_classes()[1])
```




    [[1, ()], [1, ()], [1, ()], [1, (1,2)], [1, (1,2)], [1, (1,2)]]



$3*tr_{\pi_{i}}(e) + 3*tr_{\pi_{i}}(1,2)$

From the character table for $Z_{2}$, and labelling each excitation 

$\begin{matrix}
 1 & -1 & tr_{\pi_{2}} &-> &E\\
 1 & 1  & tr_{\pi_{1}} &-> &D\\
\end{matrix}$ 

Therefore $D$ condenses on the boundary as the inner product is greater than zero, the others go to zero. 


```python
inner_product_of_characters(QDM_S3, QDM_S3.subgroups()[5], QDM_S3.conjugacy_classes()[2])
```




    [[1, ()], [1, ()], [1, (1,2,3)], [1, (1,3,2)], [1, (1,3,2)], [1, (1,2,3)]]



$2*tr_{\pi_{i}}(e) + 2*tr_{\pi_{i}}(1,2,3) + 2*tr_{\pi_{i}}(1,3,2)$

From the character table for $Z_{3}$, and labelling each excitation 

$\begin{matrix}
 1   & 1            & 1        &tr_{\pi_{1}} &-> &F \\
 1   & zeta3        &-zeta3- 1      &tr_{\pi_{2}} &-> &G \\
 1   & -zeta3 - 1   & zeta3    &tr_{\pi_{3}} &-> &H
\end{matrix}$ 

Therefore $F$ condenses on the boundary as the inner product is greater than zero, the others go to zero. 

Hence, for the subgroup $ K = G $, the excitations $ A, D, F $ condense on the boundary.

 * * * 

Similarly varying the boundaries (different subgroups) and using the inner product, the excitations which condense on the boundary can be determined.


```python
def boundary_condensates(QDM_group, QDM_subgroup):
    total_inner_product_terms = []
    for conj_class in QDM_group.conjugacy_classes():
        total_inner_product_terms.append(inner_product_of_characters(QDM_group, QDM_subgroup, conj_class))
    return total_inner_product_terms
```

<style TYPE="text/css">
code.has-jax {font: inherit; font-size: 100%; background: inherit; border: inherit;}
</style>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {
        inlineMath: [['$','$'], ['\\(','\\)']],
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'] // removed 'code' entry
    }
});
MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

Boundary condensates for the boundary indexed by $\{e, \tau\}$


```python
boundary_condensates(QDM_S3, QDM_S3.subgroups()[1])
```




    [[[3, ()], [1, (1,2)], [1, (2,3)], [1, (1,3)]],
     [[1, ()], [1, ()], [1, ()], [1, (1,2)], [1, (1,2)], [1, (1,2)]],
     []]



Observing the character table list, $ A, C, D $ condense given the boundary is indexed by $\{e, \tau\}$

* * * 

#### Construction of the ribbon operators for lattice with boundary

Given that the boundary is given by the boundary (subgroup $K$), the ribbon operator with an excitation in the bulk and the condensate on the boundary is given by

$T^{(k,g)} = \varSigma_{l \in K} F^{(lkl^{-1}, gl^{-1})}$ where $k \in K, g \in G$ 

Fixing the subgroup $K = \{e, \tau\}, (\{e, (2,3)\}$ for example)


```python
K = QDM_S3.subgroups()[1];K
```




    Subgroup of (Symmetric group of order 3! as a permutation group) generated by [(2,3)]




```python
def ribbon_operator_constructs(QDM_group, subgroup):
    ribbon_operator_terms = []
    for k in subgroup:
        for g in QDM_group:
            for l in subgroup:
                ribbon_operator_terms.append([k,g,l*k*l^-1, l*g^-1])
    return ribbon_operator_terms
ribbon_operator_constructs(QDM_S3, K)
```




    [[(), (), (), ()],
     [(), (), (), (2,3)],
     [(), (1,2), (), (1,2)],
     [(), (1,2), (), (1,2,3)],
     [(), (1,2,3), (), (1,3,2)],
     [(), (1,2,3), (), (1,3)],
     [(), (1,3,2), (), (1,2,3)],
     [(), (1,3,2), (), (1,2)],
     [(), (2,3), (), (2,3)],
     [(), (2,3), (), ()],
     [(), (1,3), (), (1,3)],
     [(), (1,3), (), (1,3,2)],
     [(2,3), (), (2,3), ()],
     [(2,3), (), (2,3), (2,3)],
     [(2,3), (1,2), (2,3), (1,2)],
     [(2,3), (1,2), (2,3), (1,2,3)],
     [(2,3), (1,2,3), (2,3), (1,3,2)],
     [(2,3), (1,2,3), (2,3), (1,3)],
     [(2,3), (1,3,2), (2,3), (1,2,3)],
     [(2,3), (1,3,2), (2,3), (1,2)],
     [(2,3), (2,3), (2,3), (2,3)],
     [(2,3), (2,3), (2,3), ()],
     [(2,3), (1,3), (2,3), (1,3)],
     [(2,3), (1,3), (2,3), (1,3,2)]]



$ T^{(e,e)} = F^{(e,e)} + F^{(e,(2,3))}, \\
T^{(e,(1,2))} = F^{(e,(1,2))} + F^{(e,(1,2,3))},\\
T^{(e,(1,2,3))} = F^{(e,(1,3,2))} + F^{(e,(1,3))},\\
T^{((2,3),e)} = F^{((2,3),e)} + F^{((2,3),(2,3))},\\
T^{((2,3),(1,2))} = F^{((2,3),(1,2))} + F^{((2,3),(1,2,3))},\\
T^{((2,3),(1,2,3))} = F^{((2,3),(1,3,2))} + F^{((2,3),(1,3))}$

Similarly for various boundaries, various ribbon operators connecting the bulk to the boundary can be generated. It is observed that for every boundary (every subgroup) there are 6 unique ribbon operators connecting the bulk to boundary.

* * *

##### Ground states with respect to different `T` operators on a cylinder with a single lattice (implying boundary on both sides of the lattice)

The lattice looks in the following way : 

$\begin{matrix}
-&-&g_{1}&-&-&-&-&-&-&g_{1}&-&-\\      
 & &     & & & | & & &     & & \\
 & &     & & & | & & &     & & \\
 & &     & & & g_{2} & & &     & & \\
 & &     & & & | & & &     & & \\
 & &     & & & | & & &     & & \\ 
-&-&g_{3}&-&-&-&-&-&-&g_{3}&-&-
\end{matrix}$

Eigenstates of $\Pi \{\varSigma$ $(vertex$ $operators)\}$ $(face$ $operators) T$ are the ground states of the lattice with an operator. In the above lattice $g_{1} and g_{3}$ are restricted to the subgroup (identified as boundary). There are three conditions to be satisfied, fixing the boundary to be $\{e, \tau\}$, due to the ribbon operators $g_{2}$ is restricted to $\{e, (2,3)\}$, due to the face operators the relationship between $g_{1}, g_{2}, g_{3}$ is as follows $g_{3}g_{2}g_{1}g_{2}^{-1} = e$, and finally due to the vertex operators $g_{1}, g_{2}, g_{3}$ get mapped to $k_{u}g_{1}k_{u}^{-1}, k_{d}g_{2}k_{u}^{-1} , k_{d}g_{3}k_{d}^{-1}$ respectively, where $k_{u}, k_{d} \in K$ 


```python
def ground_state_terms(g1, g2, g3, ku,  kd):
    return ku*g1*ku^-1, kd*g2*ku^-1, kd*g3*kd^-1
```


```python
def ground_state_sum(condition_set, subgroup):
    s = []
    for g2 in condition_set[1]:
        for g3 in subgroup:
            for g1 in subgroup:
                if condition_set[0]*g3*g2*condition_set[0]*g1 == g2:
                    s.append((condition_set[0]*g1,g2,condition_set[0]*g3))
                    for i in subgroup:
                        for j in subgroup:
                            s.append([ground_state_terms(condition_set[0]*g1, g2, condition_set[0]*g3, i, j)])
    return s
```

Observing that $ T^{(e,e)} = F^{(e,e)} + F^{(e,(2,3))}$ the condition set is that $g_{2} \in \{e,(2,3)\}$ similarly to determine the other ground states the condition set is required


```python
ground_state_sum([QDM_S3[0],[QDM_S3[0], QDM_S3[4]]], QDM_S3.subgroups()[1])
```




    [((), (), ()),
     [((), (), ())],
     [((), (2,3), ())],
     [((), (2,3), ())],
     [((), (), ())],
     ((2,3), (), (2,3)),
     [((2,3), (), (2,3))],
     [((2,3), (2,3), (2,3))],
     [((2,3), (2,3), (2,3))],
     [((2,3), (), (2,3))],
     ((), (2,3), ()),
     [((), (2,3), ())],
     [((), (), ())],
     [((), (), ())],
     [((), (2,3), ())],
     ((2,3), (2,3), (2,3)),
     [((2,3), (2,3), (2,3))],
     [((2,3), (), (2,3))],
     [((2,3), (), (2,3))],
     [((2,3), (2,3), (2,3))]]



This implies for the operator $T^{(e,e)}$ :

$\begin{matrix}
Possible \hspace{2 mm} initial \hspace{2 mm} configuration & Ground  \hspace{2 mm} state \\
(e,e,e) &  2*(e,e,e) + 2*(e,(2,3),e) \\
((2,3), e, (2,3)) & 2*((2,3), e, (2,3)) + 2*((2,3), (2,3), (2,3)) \\
(e, (2,3), e) & 2*(e,e,e) + 2*(e,(2,3),e) \\
((2,3), (2,3), (2,3)) & 2*((2,3), e, (2,3)) + 2*((2,3), (2,3), (2,3))
\end{matrix}$


```python
ground_state_sum([QDM_S3[0],[QDM_S3[1], QDM_S3[2]]], QDM_S3.subgroups()[1])
```




    [((), (1,2), ()),
     [((), (1,2), ())],
     [((), (1,2,3), ())],
     [((), (1,3,2), ())],
     [((), (1,3), ())],
     ((), (1,2,3), ()),
     [((), (1,2,3), ())],
     [((), (1,2), ())],
     [((), (1,3), ())],
     [((), (1,3,2), ())]]



This implies for the operator $T^{(e,(1,2))}$ :

$\begin{matrix}
Possible \hspace{2 mm} initial \hspace{2 mm} configuration & Ground  \hspace{2 mm} state \\
(e,(1,2),e) &  (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) \\
(e,(1,2,3),e) & (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) 
\end{matrix}$


```python
ground_state_sum([QDM_S3[0],[QDM_S3[3], QDM_S3[5]]], QDM_S3.subgroups()[1])
```




    [((), (1,3,2), ()),
     [((), (1,3,2), ())],
     [((), (1,3), ())],
     [((), (1,2), ())],
     [((), (1,2,3), ())],
     ((), (1,3), ()),
     [((), (1,3), ())],
     [((), (1,3,2), ())],
     [((), (1,2,3), ())],
     [((), (1,2), ())]]



This implies for the operator $T^{(e,(1,2,3))}$ :

$\begin{matrix}
Possible \hspace{2 mm} initial \hspace{2 mm} configuration & Ground  \hspace{2 mm} state \\
(e,(1,3),e) &  (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) \\
(e,(1,3,2),e) & (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) 
\end{matrix}$


```python
ground_state_sum([QDM_S3[4],[QDM_S3[0], QDM_S3[4]]], QDM_S3.subgroups()[1])
```




    [((2,3), (), (2,3)),
     [((2,3), (), (2,3))],
     [((2,3), (2,3), (2,3))],
     [((2,3), (2,3), (2,3))],
     [((2,3), (), (2,3))],
     ((), (), ()),
     [((), (), ())],
     [((), (2,3), ())],
     [((), (2,3), ())],
     [((), (), ())],
     ((2,3), (2,3), (2,3)),
     [((2,3), (2,3), (2,3))],
     [((2,3), (), (2,3))],
     [((2,3), (), (2,3))],
     [((2,3), (2,3), (2,3))],
     ((), (2,3), ()),
     [((), (2,3), ())],
     [((), (), ())],
     [((), (), ())],
     [((), (2,3), ())]]



This implies for the operator $T^{((2,3),e)}$ :

$\begin{matrix}
Possible \hspace{2 mm} initial \hspace{2 mm} configuration & Ground  \hspace{2 mm} state \\
(e,e,e) &  2*(e,e,e) + 2*(e,(2,3),e) \\
((2,3), e, (2,3)) & 2*((2,3), e, (2,3)) + 2*((2,3), (2,3), (2,3)) \\
(e, (2,3), e) & 2*(e,e,e) + 2*(e,(2,3),e) \\
((2,3), (2,3), (2,3)) & 2*((2,3), e, (2,3)) + 2*((2,3), (2,3), (2,3))
\end{matrix}$


```python
ground_state_sum([QDM_S3[4],[QDM_S3[1], QDM_S3[2]]], QDM_S3.subgroups()[1])
```




    [((), (1,2), ()),
     [((), (1,2), ())],
     [((), (1,2,3), ())],
     [((), (1,3,2), ())],
     [((), (1,3), ())],
     ((), (1,2,3), ()),
     [((), (1,2,3), ())],
     [((), (1,2), ())],
     [((), (1,3), ())],
     [((), (1,3,2), ())]]



This implies for the operator $T^{((2,3),(1,2))}$ :

$\begin{matrix}
Possible \hspace{2 mm} initial \hspace{2 mm} configuration & Ground  \hspace{2 mm} state \\
(e,(1,2),e) &  (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) \\
(e,(1,2,3),e) & (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) 
\end{matrix}$


```python
ground_state_sum([QDM_S3[4],[QDM_S3[3], QDM_S3[5]]], QDM_S3.subgroups()[1])
```




    [((), (1,3,2), ()),
     [((), (1,3,2), ())],
     [((), (1,3), ())],
     [((), (1,2), ())],
     [((), (1,2,3), ())],
     ((), (1,3), ()),
     [((), (1,3), ())],
     [((), (1,3,2), ())],
     [((), (1,2,3), ())],
     [((), (1,2), ())]]



This implies for the operator $T^{((2,3),(1,2,3))}$ :

$\begin{matrix}
Possible \hspace{2 mm} initial \hspace{2 mm} configuration & Ground  \hspace{2 mm} state \\
(e,(1,3,2),e) &  (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) \\
(e,(1,3),e) & (e,(1,2),e) + (e,(1,2,3),e) + (e,(1,3,2),e) + (e,(1,3),e) 
\end{matrix}$

Therefore, there are 3 unique ground states for all possible configurations of ribbon operators with an excitation at one end and condensate at the other