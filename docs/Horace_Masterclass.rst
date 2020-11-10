##################
Horace Masterclass
##################

 The below is a Matlab script file that was written for a masterclass on the use of Horace, given by R. A. Ewings on 29th April 2010. All of the relevant data files can be found on the Melehan machine, and a copy of this scrip file is held in ``/sandata/home/merlin01/users/Demostration/``.

::

   %==========================================================================
   %===== Making an sqw file from lots of spe files ==========================
   %==========================================================================

   indir='/sandata/home/sqs43493/MnSi/March2010/Data/';     % source directory of spe files
   par_file='/sandata/home/sqs43493/MnSi/March2010/Data/one2one_095.par';     % detector parameter file
   sqw_file='/sandata/sqs43493/MnSi/March2010/horace_demo.sqw';        % output sqw file

   efix=176.26;%Ei (as calculated by Homer)
   emode=1;%direct geometry
   alatt=[4.556,4.556,4.556];%lattice parameters
   angdeg=[90,90,90];
   u=[1,1,0];%u=// to incident beam
   v=[0,0,1];
   omega=0;dpsi=0;gl=0;gs=0;%set to zero here, but can be non zero if crystal was not correctly aligned
   %see the horace online manual for definitions of these angles.

   %Now check which spe files exist, starting from the run number specified by
   %run_start.

   irun=[7663:7682 7684:7696 7703:7704 7705:7727];
   psi=[-55:2:-17 -13:2:11 -15 13:2:59];

   nfiles=numel(irun);

   %make cell arrays required by Horace functions:
   spe_file=cell(1,nfiles);
   tmp_file=cell(1,nfiles);
   for i=1:nfiles
       spe_file{i}=[indir,'MER0',num2str(irun(i)),'_one2one_095.spe'];
       tmp_file{i}=[indir,'MER0',num2str(irun(i)),'_one2one_095.tmp'];
   end

   %============
   %Now make the sqw file:
   gen_sqw (spe_file, par_file, sqw_file, efix, emode, alatt, angdeg,u, v, psi,...
       omega, dpsi, gl, gs);
   %This almost always works. However sometimes you can get errors between the
   %creation of "tmp" files and the master "sqw". If so you can break the
   %tasks done by gen_sqw into smaller chunks:

   %Calculate the extent of a hyper-cuboid in 4-dimensional (Q,E) space that
   %would encompass a dataset for which psi goes from -55 to +90 degrees. This
   %bit must be modified if you intend to use a different psi range.
   psi_full=[-55:90];%full range of angles that you might cover with data
   urange_full = calc_sqw_urange(efix,emode,-18,162,par_file,alatt,angdeg,...
       u,v,psi_full,omega,dpsi,gl,gs);
   %
   %Now convert spe file to tmp files. Only do conversion if tmp file does not
   %already exist (saves times).
   for i=1:nfiles
       if ~exist(tmp_file{i})
	   psi_rad=psi(i)pi/180;
	   write_spe_to_sqw (spe_file{i}, par_file, tmp_file{i}, efix, emode, alatt, angdeg,...
		   u, v, psi_rad, omega, dpsi, gl, gs, [50,50,50,50], urange_full)
       end
   end

   %Finally, combine all the tmp files into an sqw file.
   write_nsqw_to_sqw(tmp_file,sqw_file);

   %==========================================================================
   %===== Fake dataset creation (for planning an experiment) =================
   %==========================================================================

   %A quick and dirty method for working out what range of angles you might
   %want to use for an experiment. Makes a fake dataset with a uniform
   %selection of angles from the specified range. You can then take cuts from
   %this dataset (see below) to work out if it covers the (Q,w) space that you
   %want.

   psi_min=0; psi_max=90;
   fake_data('/sandata/home/merlin01/users/Demonstration/',par_file,'fake_demo.sqw',efix,emode,alatt,angdeg,u,v,psi_min,psi_max,...
       omega,dpsi,gl,gs)


   %==========================================================================
   %===== Taking n-dimensional cuts from the data, and plotting them =========
   %==========================================================================

   %First let us take a diffraction cut of the fake data in order to see our
   %detector coverage of Q-space:
   fake_source='/sandata/home/merlin01/users/Demonstration/fake_demo.sqw';
   proj.u=[1,1,0]; proj.v=[-1,1,0]; proj.uoffset=[0,0,0,0]; proj.type='rrr';
   test_diffraction=cut_sqw(fake_source,proj,[-5,0.1,5],[-0.2,0.2],[-5,0.1,5],[-10,10],'-nopix');
   plot(test_diffraction);

   data_source='/sandata/sqs43493//MnSi/March2010/MnSi_300K_80meV.sqw';
   %Take a 2d cut with -nopix option
   proj.u=[1,0,0]; proj.v=[0,1,0]; proj.uoffset=[0,0,0,0]; proj.type='rrr';
   d2dcut=cut_sqw(data_source,proj,[-2,0.05,2],[-0.2,0.2],[1.8,2.2],[0,1,80],'-nopix');%creates d2d object
   plot(smooth(compact(d2dcut)));
   lz 0 1
   keep_figure;

   %Repeat, but keep pixels:
   sqwcut=cut_sqw(data_source,proj,[-2,0.05,2],[-0.2,0.2],[1.8,2.2],[0,1,80]);%creates sqw object
   plot(compact(sqwcut));%note sqw objects CANNOT be smoothed.
   lz 0 1
   keep_figure;

   d2dlook=get(d2dcut);%convert to structure array in order to use Matlab to inspect
   sqwlook=get(sqwcut);%note the extra "pix" field in sqw object

   %Take a 3d cut and use sliceomatic to plot it:
   d3dcut=cut_sqw(data_source,proj,[-2,0.05,2],[-2,0.05,2],[1.8,2.2],[0,0,80],'-nopix');
   plot(smooth(d3dcut,[3 3 3],'gaussian'));

   %Use a neat tool to quickly check diffraction:
   diffractioncut=cut_sqw(data_source,proj,[-6,0.1,6],[-6,0.1,6],[-4,0.1,10],[-5,5],'-nopix');
   sliceomatic_overview(diffractioncut);%the same as normal sliceomatic, but camera position is automatically overhead.

   %You can take a cut from a cut, e.g. a 1d cut from a 2d slice:
   sqw1dcut1=cut(sqwcut,[],[20,30]);
   sqw1dcut2=cut(sqwcut,[],[30,40]);

   %notice we can use Libisis/mgenie style plot commands for 1d datasets (d1d
   %or sqw).
   acolor black
   plot(sqw1dcut1);
   acolor red
   pp(sqw1dcut2);





   %==========================================================================
   %===== Graphical user interface (GUI) =====================================
   %==========================================================================

   %Start it up:
   horace
   %and away you go!

   %==========================================================================
   %===== Dealing with error messages ========================================
   %==========================================================================

   %Let's deliberately do something wrong:
   d3dcut=cut_sqw(data_source,proj,[-2,0.05,2],[-2,0.05,2],[1.8,2.2],[0,1,80],'-badger');
   %
   d3dcut=cut_sqw(data_source,proj,[3,0.05,2],[-2,0.05,2],[1.8,2.2],[0,1,80],'-nopix');
   %lots of red writing on the screen, but notice that the error message is
   %one that we wrote, and gives some clue as to what you did wrong...
   %
   % for further info on how a function works you can type, for example
   help cut_sqw

   %or look on the website: http://horace.isis.rl.ac.uk

   %==========================================================================
   %===== Background subtraction =============================================
   %==========================================================================

   %Several different ways of doing this:

   %1) Subtract a number from the data (flat background):
   testminus_sqw=minus(sqwcut,0.1);%subtract 0.1 from all pixels
   testminus_d2d=minus(d2dcut,0.1);%does the same thing on a d2d data object

   %You can do other binary operations, e.g. divide (mrdivide - matrix right divide),
   %times (mtimes - matrix times), plus, power (mpower - matrix power).

   %=====
   %2) Subtract one object from another, where one is the background and the
   %other is the data:

   d1dcut1=d1d(sqw1dcut1);%notice that we can convert an sqw object to a dnd object - this
   %simply means we throw away the detector pixel info. We can do the
   %opposite, but we create an sqw object that has no pixel info.
   d1dcut2=d1d(sqw1dcut2);
   %
   testminus_d1d=minus(d1dcut1,d1dcut2);
   plot(testminus_d1d);


   %=====
   %3) Increase the dimensionality of a cut taken in a specific part of
   %reciprocal space so that it can be subtracted from a larger section of the
   %data:

   bg_1d=cut(d2dcut,[-0.1,0.1],[]);%take a 1d cut along energy axis
   bg_2d=replicate(bg_1d,d2dcut);%syntax is lower dimensional cut 1st, then reference object 2nd
   bg_subtracted_data=minus(d2dcut,bg_2d);
   plot(bg_subtracted_data); lz 0 1


   %==========================================================================
   %===== Symmetrisation =====================================================
   %==========================================================================

   %For this to work we need to use sqw objects, since we need to know all of
   %the pixel information. There are 2 options, either use symmetrisation to
   %work on an existing cut (most common option), or make a new sqw file that
   %combines specified equivalent Brillouin zones (more complicated and not
   %always appropriate).

   nice_hk=cut_sqw(data_source,proj,[-2,0.05,2],[-2,0.05,2],[1.8,2.2],[10,15]);
   plot(nice_hk); lz 0 2; keep_figure;
   %
   nice_hk_sym1=symmetrise_sqw(nice_hk,[0,0,1],[1,1,0],[0,0,2]);
   plot(nice_hk_sym1); lz 0 2; keep_figure;
   %
   nice_hk_sym2=symmetrise_sqw(nice_hk_sym1,[0,0,1],[-1,1,0],[0,0,2]);
   plot(nice_hk_sym2); lz 0 2; keep_figure;
   %
   %Look at error bars:
   cut_from_nice=cut(nice_hk,[0.9,1.1],[]);
   cut_from_nice_ortho=cut(nice_hk,[-0.1,0.1],[]);
   cut_from_nice_sym1=cut(nice_hk_sym1,[0.9,1.1],[]);
   cut_from_nice_sym2=cut(nice_hk_sym2,[0.9,1.1],[]);
   acolor red
   plot(cut_from_nice);
   acolor blue
   pp(cut_from_nice_sym1);
   acolor black
   pp(cut_from_nice_sym2);
   keep_figure;

   acolor green
   plot(cut_from_nice_ortho);
   %see a slight improvement, probably because the errorbars for the two rings
   %nearer the edge had larger errorbars to start with...

   %=====
   %There is also the (brand new) possibility of creating an sqw file that
   %is centered on one Brillouin zone, and combines data from other specified
   %equivalent zones. This is presently a rather ugly piece of code and
   %requires vast amounts of memory to run, hence it is realistically only
   %possible to run it on Melehan at present.


   pos=[-1,0,2];
   step=0.05;
   outfile='/sandata/home/merlin01/users/Demonstration/test_sym.sqw';
   erange=[0,0,72];

   wout=combine_equivalent_zones(data_source,proj,pos,step,erange,outfile,'-ab');

   %Here's one I made earlier...
   symdata1='/sandata/sqs43493/MnSi/March2010/test_80meV_sym.sqw';
   symcut1=cut_sqw(symdata1,proj,[-2,0.05,0],[-0.1,0.1],[1.9,2.1],[20,30]);
   datacut1=cut_sqw(data_source,proj,[-2,0.05,0],[-0.1,0.1],[1.9,2.1],[20,30]);

   acolor red
   plot(datacut1);
   acolor blue
   pp(symcut1);
   ly 0.2 0.8



   %==========================================================================
   %===== Simulations ========================================================
   %==========================================================================

   %We can use Horace to simulate a given S(Q,w) model for the same range as a
   %specified cut. Or we can use more simple functions (e.g. gaussian peaks
   %etc).

   sqw_params=[10,0.035,50,29,10,4.551,0,0];
   simulated_sqw=sqw_eval(d2dcut,@bg_MK_fluctuations_sqw,sqw_params);
   plot(simulated_sqw); lz 0 1; keep_figure;
   func_params=[0.25,0.25,-1,1,0.3,0.3,0.35];
   simulated_func=func_eval(d1dcut1,@two_gauss,func_params);
   acolor blue
   plot(d1dcut1);
   acolor red
   pl(simulated_func);

   %NOTE - IF WE NEED TO PASS MORE INFORMATION TO THE FUNCTIONS, E.G. LOOKUP
   %TABLES OF FORM FACTORS, THEN WE CAN DO THIS BY MAKING SQW_PARAMS OR
   %FUNC_PARAMS A CELL ARRAY, WITH THE FIRST ELEMENT A VECTOR.
   %e.g:
   sqw_parms={[10,0.035,50,29,10,4.551,0,0],info1,info2};

   %==========================================================================
   %===== Basic fitting ======================================================
   %==========================================================================

   %Similarly to simulation, we can also do fits for one or more cuts. We can
   %do the usual thing of having some parameters free and some held fixed, and
   %we can also bind pairs of parameters together in a specified ratio.

   %Look at some real data now:
   load('/sandata/home/sqs43493/MnSi/March2010/Analysis/OldNewData_comparison_workspace_13April.mat','Ei80_1d_barhh_final');

   guess_pars=[10,0.035,50,29,10,4.551,0,0];
   free_pars=[1,0,1,0,0,0,0,0];
   bgpars=[0.4,-0.05];
   bgfree=[1,1];
   [wfit,fitdata]=fit_sqw(Ei80_1d_barhh_final([1,2,4,5,7]),@bg_MK_fluctuations_sqw,guess_pars,free_pars,...
       @linear_bg,bgpars,bgfree,'list',2,'fit',[0.001,50,0.001]);

   cuts_chosen=[1,2,4,5,7];
   for i=1:5
       acolor blue
       plot(Ei80_1d_barhh_final(cuts_chosen(i)));
       acolor red
       pl(wfit(i));
       keep_figure;
   end

   %similar syntax is used with fit_func, for which we have a function that is
   %not an S(Q,w) model.

   %==========================================================================
   %===== Advanced fitting (multifit) ========================================
   %==========================================================================

   %One of the most powerful, but under-utilised, functions in Horace is
   %multifit. This works similarly to normal fitting, but a series of cuts are
   %all fitted simultaneously to the same model and parameter set (although
   %backgrounds are kept independent). This is very much like the kind of
   %thing you can do with Tobyfit, although at present we do not have the
   %possibility of resolution convolution

   %The example we show here is a rather advanced one, involving a complicated
   %set of parameter bindings.

   %The parameters for each cut that guess the actual background:
   slope_pars={[0.4,-0.05],[0.5,-0.17],[0.42,0],[0.3,0],[0.2,-0.02]};

   %The initial guess parameters for the spectral function:
   specpars=cell(1,5);
   for i=1:5
       specpars{i}=[12 0.035 55 29 11 4.551];
   end

   %Combine the above in a set of "background" parametrs.
   bgpars_multifit=cell(1,5); bgfix_multifit=cell(1,5);
   for i=1:5
       bgpars_multifit{i}=[specpars{i} slope_pars{i}];
       bgfix_multifit{i}=[1 1 1 0 0 0 1 1];
   end

   %Specify a section of 1 cut to ignored during fitting:
   removerange=cell(1,5);
   removerange{2}=[-0.7,0];

   %Specify bindings:
   bpbind=cell(1,5);
   bpbind{1}={ {3,2,0,1},{2,1,0,1} };%parameter 3 of "background" bound to parameter 2 of null function etc.
   for i=2:5
       bpbind{i}={ {3,2,0,1},{2,1,0,1},{1,1,1,1} };%as above, but parameter 1 of all bg functions
       %bound to paramter 1 of 1st background function
   end

   %Now we're ready to fit!
   [wmultifit,multifitdata]=multifit_sqw_sqw(Ei80_1d_barhh_final([1,2,4,5,7]),...
       @nullfunc,...
       [0.0325 55],[0,1],...
       @bg_MK_fluctuations_sqw,bgpars_multifit,bgfix_multifit,...
       bpbind,'fit',[0.001 50 0.001],'list',1,'remove',removerange);

   cuts_chosen=[1,2,4,5,7];
   for i=1:5
       acolor blue
       plot(Ei80_1d_barhh_final(cuts_chosen(i)));
       acolor red
       pl(wmultifit(i));
       keep_figure;
   end
