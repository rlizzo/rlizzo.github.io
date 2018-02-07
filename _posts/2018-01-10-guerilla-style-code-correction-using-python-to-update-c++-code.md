---
layout: post
title:  "Guerilla Style Code Correction - Using Python to Update C++ Code"
date:   2018-01-10
excerpt: "How did this actually work?"
image: "/images/posts/2018-01-10/cpppython.jpeg"
published: true
---

In our recent effort to modernize the VMTK build and distribution system I came across a series of issues which reminded me why I so love programming with the ever adaptable Python language. What I thought would be an inconsequential change in our build/packaging system, updating a single line of CMake code `CMAKE_CXX_STANDARD=98` -> `CMAKE_CXX_STANDARD=11`, turned our error free build system into the stuff of compile time nightmares. 

 In C++11, a new keyword was added to the language: `override`. Any time that your code wishes to override a method from a derived class, you are now supposed to include the `override` keyword in order to make your intentions clear to the compiler. That way, if you misstype the name of a derived class, the compiler can issue an error letting you know there is no such method to override. It seems like its a beneficial language feature, and it will probably be very useful to me in the future; however, it also meant that any time our (outdated) code didn't specify that magic `override` keyword, the compiler would kindely let us know by issuing a warning...

 Actually, more like 600+ of them... 

```python
cwd = os.getcwd()
p = Path('/Users/rick/projects/vmtk/vmtk/vtkVmtk/')
header_files = list(p.glob('**/vtkvmtk*.h'))

no_replace = ['ComputeCenterlineSplitting', 
             'ComputePointAndGapCenterlineSplitting', 
             'ComputeBetweenPointsCenterlineSplitting',
             'MergeTracts',
             'GroupTracts',
             'SetTargetSeedIds',
             'SetSourceSeedIds',
             'SetCapCenterIds',
             'VTK_OVERRIDE',
             'Apply',
             'CanReadFile',
             '=']

replace_start = ['void PrintSelf',
                'void WriteData',
                'virtual int RequestData',
                'double EvaluateFunction',
                'virtual double EvaluateFunction',
                'virtual void SimpleExecute',
                'vtkMTimeType GetMTime',
                'virtual void CopyParameters',
                'int FillInputPortInformation',
                'virtual void DeepCopy',
                'virtual vtkvmtkItem* InstantiateNewItem',
                'virtual TimeStepType',
                'virtual void Build()',
                'virtual int FillInputPortInformation',
                'virtual void SetLastCellId',
                'void InternalUpdate',
                'void EvaluateGradient',
                'virtual int RequestInformation',
                'virtual int FunctionValues',
                'virtual void Build() = 0']

filenames_for_build = ['vtkvmtkDataSetItem.h',
                      'vtkvmtkFEAssembler.h']

# need to keep vtkvmtkNeighborhood.h', 'vtkvmtkStencil.h',
```

```python
for file in header_files:
    for line in fileinput.input(str(file), inplace=1): 
        change_line = 1
        if line.strip().endswith(';'):
            if line.count(';') > 1:
                change_line = 0
        if file.name == 'vtkvmtkItem.h':
            if line.strip().startswith('virtual void DeepCopy'):
                change_line = 0                
        for text in no_replace:
            if text in line:
                if line.strip().startswith('virtual void Build() = 0'):
                    change_line = 1
                else:
                    change_line = 0
        if file.name in filenames_for_build:
            if line.strip().startswith('virtual void Build() = 0'):
                change_line = 0

                
        if change_line == 0:
            pass
        else:
            for text in replace_start:
                if line.strip().startswith('virtual void Build() = 0'):
                    line = line.replace('Build() =', 'Build() VTK_OVERRIDE =')
                    break
                elif line.strip().startswith(text):
                    line = line.replace(';', ' VTK_OVERRIDE;')
                    break
        sys.stdout.write(line)
```


